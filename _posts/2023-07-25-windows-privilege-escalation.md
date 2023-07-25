---
title: '01 - Introduction to Windows Privilege Escalation'
date: 2023-07-25 17:49:22 +0700
categories: ['Windows Hacking', 'Windows Privilege Escalation']
tags: [red team, introduction, what is, privilege escalation, windows hacking, windows privilege escalation]     # TAG names should always be lowercase
author: nairpaa
---

**Host Privilege Escalation** memungkinkan kita untuk meningkatkan hak akses dari pengguna standar menjadi Administrator. Ini bukan langkah yang mutlak diperlukan, karena ada cara lain untuk memperoleh kredensial dan *lateral movement* di dalam domain tanpa perlu melakukan *privilege escalation* terlebih dahulu.

Namun, dengan mendapatkan hak akses Administrator memberikan keuntungan taktis dengan memungkinkan kita memanfaatkan beberapa kemampuan tambahan. Misalnya, melakukan dump kredensial dengan `Mimikatz`, menginstal persistensi yang sulit terdeteksi, atau memanipulasi konfigurasi host seperti firewall.

Selaras dengan prinsip "*principle of least privilege*" - *privilege escalation* hanya harus dicari jika hal itu memberikan cara untuk mencapai tujuan kita, bukan sesuatu yang kita lakukan "*just because*". 

Mengeksploitasi kerentanan *privilege escalation* memberikan data tambahan kepada *defenders* untuk mendeteksi keberadaan kita. Ini adalah kalkulasi *risk* vs *reward* yang harus kita lakukan.

Metode umum untuk *privilege escalation* mencakup kesalahan konfigurasi sistem operasi atau software pihak ketiga dan *missing patches*.

[SharpUp](https://github.com/GhostPack/SharpUp) dapat mengenumerasi host untuk melihat peluang kerentanan *misconfiguration-based*.

## 0x1 - Table of Contents

**Misconfiguration-based**:
1. [[Misconfig] What is Windows Service?](/posts/windows-service/)
2. [[Misconfig] Unquoted Service Paths](/posts/unquoted-service-paths/)
3. [[Misconfig] Weak Service Permissions](/posts/windows-weak-service-permission/)
4. [[Misconfig] Weak Service Binary Permissions](/posts/windows-weak-binary-permission/)
5. [[Misconfig] UAC Bypass](/posts/uac-bypass/)
6. [[Privilege Abuse] SeImpersonatePrivilege or SeAssignPrimaryToken (Potatoes)](/posts/seimpersonateprivilege-or-seassignprimarytoken/)
7. [[Privilege Abuse] SeBackupPrivilege](/posts/sebackupprivilege-abuse/)

**CVE**:
1. [[CVE-2021-1675 / CVE-2021-34527] PrintNightmare](/posts/printnightmare-cve-2021-1675/)

