---
title: '[Windows] Processes'
date: 2023-07-25 00:33:22 +0700
categories: ['Windows Hacking', 'Host Reconnaissance']
tags: [red team, host reconnaissance, windows hacking, ps]     # TAG names should always be lowercase
author: nairpaa
---

Untuk melihat daftar proses yang sedang berjalan pada sistem Windows, kita bisa menggunakan perintah `ps` seperti berikut:

```powershell
beacon> ps

PS> ps
```

> Jika user yang kita jalankan saat ini tidak berjalan sebagai administrator, kita tidak akan dapat melihat informasi dari proses yang bukan milik user tersebut.
{: .prompt-warning }

Ada beberapa proses yang bisa kita lihat dengan menggunakan perintah ini, seperti kita mengetahui bahwa sistem menjalankan sistem keamanan seperti `Sysmon64`, `MsMpEng`, `elastic-endpoint`, dan `elastic-agent`.



