---
title: '[Windows] User Sessions'
date: 2023-07-25 17:06:22 +0700
categories: ['Windows Hacking', 'Host Reconnaissance']
tags: [red team, host reconnaissance, windows hacking, cobalt strike, clipboard]     # TAG names should always be lowercase
author: nairpaa
---

Pengguna lain yang saat ini sedang masuk ke dalam komputer yang sama mungkin merupakan target yang bagus untuk diserang. Jika mereka memiliki *privilege* yang lebih tinggi dari pengguna yang kita gunakan saat ini, mereka mungkin merupakan kandidat yang baik untuk melakukan *lateral movement*.

Perintah `net logons` akan mendaftarkan *logon sessions* pada komputer.

```bash
beacon> net logons

Logged on users at \\localhost:

DEV\mtyson
DEV\bmalik
DEV\WKSTN-2$
```
