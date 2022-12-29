---
title: Part 3 - Domain Enumeration with Bloodhound
date: 2022-02-19 12:04:22 +0700
categories: [Active Directory, Post Compromise Enumeration]
tags: [active directory, windows, bloodhound, enumeration]     # TAG names should always be lowercase
author: nairpaa
---

### Tujuan 

- *Post compromise enumeration*

### Prasyarat

- Memiliki kredensial user domain

### Tools

- [Bloodhound / SharpHound](https://bloodhound.readthedocs.io/en/latest/data-collection/bloodhound-py.html)

--- 

## Enumerasi AD dengan Bloodhound

#### Remote BloodHound
[Python BloodHound Repository](https://github.com/fox-it/BloodHound.py) atau install `pip3 install bloodhound`.
```bash
# remote
➜ bloodhound-python -u <user> -p <password> -ns <DC Ip> -d <Domain> -c All
```

#### On Site BloodHound
```powershell
# exe binary
PS> .\SharpHound.exe -c All
PS> .\SharpHound.exe -c All --ldapusername <user> --ldappassword <password> --domain <domain> 

# powershell
PS> . .\SharpHound.ps1
PS> Invoke-BloodHound -CollectionMethod All
PS> Invoke-BloodHound -CollectionMethod All -LdapUser <user> -LdapPass <password> -domain <domain>
```

## Artikel Tips Membaca Bloodhound

- [Attacking Active Directory Permissions with BloodHound](https://stealthbits.com/blog/attacking-active-directory-permissions-with-bloodhound/)
- [Attack Mapping with BloodHound](https://stealthbits.com/blog/local-admin-mapping-bloodhound/)
- [Active Directory Exploitation Cheat Sheet](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#domain-enumeration)