---
title: '01 - Introduction to Windows Persistence'
date: 2023-07-25 17:49:22 +0700
categories: ['Windows Hacking', 'Windows Persistence']
tags: [red team, introduction, what is, Persistence, windows hacking, windows Persistence]     # TAG names should always be lowercase
author: nairpaa
---

**Persistence** adalah metode untuk mendapatkan kembali atau mempertahankan akses ke mesin yang telah terkompromi tanpa harus mengeksploitasi langkah-langkah awal kompromi dari awal lagi.

Pada [Cyber Kill Chain](/posts/what-is-red-teaming/#0x9---attack-lifecycle), *persistence* berada pada fase **Installation**.

## 0x1 - Table of Contents

**No Admin Required**:
1. [Persistence via Task Scheduler](/posts/windows-persistence-task-scheduler/)
2. [Persistence via Startup Folder](/posts/windows-persistence-startup-folder/)
3. [Persistence via Registry AutoRun](/posts/windows-persistence-registry-autorun/)
4. Persistence via COM Hijacks

**Admin Required**:
1. [Persistence via Windows Services](/posts/windows-persistence-service/)
2. [Persistence via WMI Event Subscriptions](/posts/windows-persistence-vmi-event/)

> Ingatlah bahwa terkadang `SISTEM` tidak dapat mengautentikasi ke web proxy (karena tidak termasuk *domain users*), jadi kita tidak dapat menggunakan [*egress payload*](/posts/cobalt-strike/#a-egress-listeners).
{: .prompt-warning }