---
title: '01 - Introduction to Domain Reconnaissance'
date: 2023-07-29 13:39:22 +0700
categories: ['Active Directory', 'Post Compromise Enumeration']
tags: [red team, active directory hacking, introduction, what is, fundamental, post compromise enumeration]     # TAG names should always be lowercase
author: nairpaa
---

Setelah berhasil mendapatkan [kredensial](/posts/credential-theft/) pengguna, kita melakukan *domain reconnaissance* untuk melihat di mana kita dapat memanfaatkan kredensial tersebut.

Pada bagian ini kita akan membahas beberapa informasi domain yang bisa didapatkan dari pengguna domain standar. 

> Perlu dicatat bahwa melakukan *domain recon* tidak memerlukan akses `high integrity`, dan dalam beberapa kasus (seperti `SISTEM`) dapat merugikan.
{: .prompt-warning }

## 0x1 - Table of Contents

1. Domain Recon with PowerView
2. Domain Recon with SharpView
3. Domain Recon with AD Search
4. Domain Recon with AD Module
5. Domain Recon with BloodHound
6. Another Tools for AD Enumeration