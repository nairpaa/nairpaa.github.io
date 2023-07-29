---
title: '[Credential Theft] Domain Caache Credentials'
date: 2023-07-29 11:57:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, credential theft, mimikatz, cobalt strike, dcc]     # TAG names should always be lowercase
author: nairpaa
---

**Domain Cached Credentials (DCC)** dirancang untuk situasi di mana kredensial domain diperlukan untuk *login* ke sebuah mesin, bahkan saat mesin tersebut tidak terhubung ke domain (misalnya laptop yang berpindah-pindah lokasi). 

Perangkat lokal akan menyimpan *cache* dari kredensial domain sehingga otentikasi bisa terjadi secara lokal. Namun, kredensial ini bisa diekstraksi dan di-*crack* secara offline untuk mendapatkan kredensial teks biasa.

Sayangnya, format *hash* dari DCC bukan NTLM sehingga tidak bisa digunakan dengan metode [Pass-the-Hash](/posts/pass-attack/#pass-the-hash-pth).

## 0x1 - Exploitation Stages

> Perintah ini memerlukan hak akses yang lebih tinggi.
{: .prompt-warning }

```powershell
beacon> mimikatz !lsadump::cache
```

> **OPSEC**
>
> Modul Mimikatz ini akan membuka *handle* ke *SECURITY registry hive*. 
> 
> Kita bisa menggunakan kata kunci "`Suspicious SECURITY Hive Handle`" pada Kibana untuk melihat *log* ini.
{: .prompt-danger }

## 0x2 - Crack the Hash

Untuk meng-*crack* *hash* ini dengan [hashcat](https://hashcat.net/hashcat/), kita perlu mengubahnya ke dalam format yang diharapkan. [Halaman web hashcat](https://hashcat.net/wiki/doku.php?id=example_hashes) menunjukkan kepada kita bahwa formatnya adalah `$DCC2$<iterations>#<username>#<hash>`.

```bash
➜ hashcat -m 2100 <file.hash> <wordlist.txt>
```

> Perlu diingat bahwa DCC jauh lebih lambat untuk di-*crack* dibandingkan dengan NTLM.
{: .prompt-warning }

---

## 0x3 - References

- https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-and-cracking-mscash-cached-domain-credentials
- https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/credential-access/credential-dumping/cached-domain-credentials

