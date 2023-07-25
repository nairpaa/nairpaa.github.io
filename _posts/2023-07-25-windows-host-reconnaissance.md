---
title: '01 - Windows Host Reconnaissance'
date: 2023-07-25 00:25:22 +0700
categories: ['Windows Hacking', 'Host Reconnaissance']
tags: [red team, introduction, what is, host reconnaissance, windows hacking]     # TAG names should always be lowercase
author: nairpaa
---

Sebelum melakukan tindakan *post-exploitation*, sangat penting untuk memahami kondisi sistem target saat ini. 

Setiap tindakan yang dilakukan memiliki risiko deteksi, dan tingkat risiko ini bergantung pada kapabilitas penyerang dan pertahanan yang ada. 

Oleh karena itu, sangat penting untuk mengumpulkan sebanyak mungkin informasi tentang sistem target, termasuk perangkat lunak *Antivirus (AV)* atau *Endpoint Detection & Response (EDR)*, *Windows audit policies*, *PowerShell Logging*, *Event Forwarding*, dan lain-lain.

"**Defence in Depth**" adalah konsep di mana berbagai lapisan kontrol keamanan ditempatkan di seluruh sistem atau lingkungan. Tujuannya adalah untuk memberikan redundansi, sehingga jika satu kontrol gagal, kontrol lainnya tetap berjalan. 

Dalam konteks *red teaming*, ini berarti bahwa penyerang harus siap untuk melewati beberapa lapisan keamanan.

Konsep "**Offence in Depth**" mirip dengan *Defence in Depth*, tetapi dilihat dari perspektif penyerang. Dalam hal ini, informasi yang dikumpulkan selama fase *host reconnaissance* harus digunakan untuk merencanakan dan menyesuaikan taktik serangan. 

Misalnya, jika penyerang memiliki *script* PowerShell favorit yang melakukan tugas tertentu ("`X`"), tetapi *logging* PowerShell diaktifkan pada sistem target, penyerang mungkin perlu menghindari melakukan "`X`", atau mencari cara alternatif untuk melakukannya (misalnya menggunakan .NET bukan PowerShell). 

*Offensive Security Engineer* yang baik akan memiliki berbagai alat atau metodologi untuk mencapai tujuan yang sama.


## 0x1 - Table of Contents

1. [Processes](/posts/windows-processes/)
2. [Seatbelt](/posts/seatbelt/)
3. [Screenshots](/posts/windows-screenshot/)
4. [Keylogger](/posts/windows-keylogger/)
5. [Clipboard](/posts/windows-clipboard/)
6. [User Sessions](/posts/windows-user-sessions/)


