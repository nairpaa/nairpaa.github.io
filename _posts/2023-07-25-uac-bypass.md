---
title: '[Misconfig] UAC Bypass'
date: 2023-07-25 23:22:22 +0700
categories: [Privilege Escalation, Windows Privilege Escalation]
tags: [red team, privilege escalation, what is, windows hacking, windows privilege escalation, misconfig]       # TAG names should always be lowercase
author: nairpaa
---

**User Account Control (UAC)** adalah sebuah teknologi yang ada pada Windows yang memaksa aplikasi untuk meminta persetujuan ketika meminta token akses administratif.

Mungkin kita telah mendapatkan user dengan hak akses administrator lokal pada suatu komputer, tetapi ketika menjalankan perintah untuk menambahkan user pada **Command Prompt** kita mendapatkan penolakan akses. 

Hal ini terjadi karena `cmd.exe` berjalan dalam `medium integrity`.

```powershell
C:\> net user hacker Passw0rd! /add
System error 5 has occurred.

Access is denied.

C:\> whoami /groups

Mandatory Label\Medium Mandatory Level
```

## 0x1 - Bypassing UAC 

### A. Using GUI

Jalankan program dengan "**Run as administrator**", yang akan menyebabkan prompt UAC muncul.

![UAC Prompt](/assets/img/posts/uac-bypass/1.png)
_UAC Prompt_

Hanya setelah mengklik **Yes**, Command Prompt akan memiliki hak istimewa yang bisa membuat perubahan konfigurasi sistem, karena sekarang akan berjalan dalam "`high integrity`".

```powershell
C:> whoami /groups

Mandatory Label\High Mandatory Level
```

### B. Using Cobalt Strike

```powershell
beacon> elevate uac-schtasks tcp-local
```

---

## 0x2 - References

- https://github.com/cobalt-strike/ElevateKit
- https://book.hacktricks.xyz/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control
- https://github.com/k4sth4/UAC-bypass