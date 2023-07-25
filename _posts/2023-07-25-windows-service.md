---
title: '[Misconfig] What is Windows Service?'
date: 2023-07-25 19:52:22 +0700
categories: ['Windows Hacking', 'Windows Privilege Escalation']
tags: [red team, privilege escalation, what is, windows hacking, windows privilege escalation, service exploit, misconfig]     # TAG names should always be lowercase
author: nairpaa
---


**Windows Service** adalah jenis aplikasi khusus yang biasanya dijalankan secara otomatis saat komputer melakukan *booting*.

*Service* digunakan untuk memulai dan mengelola fungsionalitas Windows *core* seperti Windows Defender, Windows Firewall, Windows Updater, dan lainnya.

Aplikasi pihak ketiga juga dapat menginstal Windows *service* untuk mengelola bagaimana dan kapan aplikasi tersebut dijalankan.

Kita dapat melihat *service* yang diinstal pada komputer dengan membuka `services.msc`, atau menggunakan perintah `sc`.

```powershell
C:\> sc query

SERVICE_NAME: Appinfo
DISPLAY_NAME: Application Information
        TYPE               : 30  WIN32
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

SERVICE_NAME: AudioEndpointBuilder
DISPLAY_NAME: Windows Audio Endpoint Builder
        TYPE               : 30  WIN32
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Dan cmdlet `Get-Service` PowerShell.

```powershell
PS > Get-Service | fl

Name                : AJRouter
DisplayName         : AllJoyn Router Service
Status              : Stopped
DependentServices   : {}
ServicesDependedOn  : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
ServiceType         : Win32ShareProcess

Name                : ALG
DisplayName         : Application Layer Gateway Service
Status              : Stopped
DependentServices   : {}
ServicesDependedOn  : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
ServiceType         : Win32OwnProcess
```

Sebuah *service* memiliki beberapa properti yang bisa kita perhatikan, seperti:

### 0x1 - Binary Path

Ini adalah PATH di mana file eksekusi (`.exe`) dari *service* tersebut berada. Windows *service* sering kali berada di `C:\Windows\system32` dan pihak ketiga di `C:\Program Files` atau `C:\Program Files (x86)`.

### 0x2 - Startup Type

Ini menentukan kapan *service* harus dimulai.

- **Automatic**: *Service* akan dijalankan segera saat booting.
- **Automatic (Delayed Start)**: *Service* menunggu sebentar setelah booting sebelum dijalankan. Ini sebagian besar adalah *legacy option* untuk membantu desktop memuat lebih cepat.
- **Manual**: *Service* hanya akan dijalankan saat diminta secara khusus.
- **Disabled**: *Service* dinonaktifkan dan tidak akan berjalan.

### 0x3 - Service Status

Ini adalah status saat ini dari *service* tersebut.

- **Running**: *Service* sedang berjalan.
- **Stopped**: *Service* tidak berjalan.
- **StartPending**: *Service* telah diminta untuk dijalankan dan sedang menjalankan prosedur *startup*-nya.
- **StopPending**: *Service* telah diminta untuk berhenti dan sedang menjalankan prosedur penutupannya.

### 0x4 - Log On As

Akun pengguna yang dikonfigurasikan untuk menjalankan *service*.

Ini bisa berupa akun domain atau lokal. Sering kali *service* ini untuk dijalankan sebagai akun dengan hak istimewa tinggi, bahkan domain admin, atau sebagai sistem lokal. Inilah sebabnya mengapa *service* dapat menjadi target yang menarik untuk eskalasi hak istimewa lokal dan domain.

### 0x5 - Dependants & Dependencies

Ini adalah *service* yang bergantung pada *service* saat ini untuk berjalan, atau *service* lain yang bergantung pada *service* ini untuk berjalan. Informasi ini penting untuk memahami dampak potensial dari manipulasi.

> **OPSEC**
> 
> Kembalikan konfigurasi *service* setelah Anda selesai. Pastikan Anda tidak mengganggu *service* penting bisnis, jadi mintalah izin sebelum mengeksploitasi jenis kerentanan ini.
{: .prompt-danger }
