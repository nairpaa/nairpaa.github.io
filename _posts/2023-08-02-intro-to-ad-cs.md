---
title: '[AD CS] 01 - Introduction'
date: 2023-08-02 01:25:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, introduction, what is, active directory hacking, windows hacking, post compromise attack, ad cs]     # TAG names should always be lowercase
author: nairpaa
---

**Active Directory Certificate Services (AD CS)** memungkinkan organisasi untuk membangun *Public Key Infrastructure* (PKI) sendiri. 

PKI ini menyediakan fungsi-fungsi keamanan seperti kriptografi kunci publik, sertifikat digital, dan kemampuan tanda tangan digital. 

Beberapa aplikasi praktis dari teknologi ini termasuk Secure/Multipurpose Internet Mail Extensions (S/MIME) untuk email terenkripsi, VPN untuk jaringan pribadi yang aman, IPsec untuk komunikasi yang aman, EFS untuk enkripsi file, serta autentikasi dengan kartu pintar dan SSL/TLS untuk komunikasi yang aman di web.

Jika diimplementasikan dengan benar, AD CS dapat meningkatkan keamanan organisasi dengan cara:
- **Kerahasiaan**: Melalui enkripsi, data dapat dilindungi dari akses yang tidak sah.
- **Integritas**: Tanda tangan digital memastikan bahwa data tidak diubah atau disalahgunakan.
- **Otentikasi**: Mengaitkan kunci sertifikat dengan akun komputer, pengguna, atau perangkat memastikan bahwa entitas dalam jaringan dapat diidentifikasi dengan benar.

> Anda bisa membaca [buku](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) bagus yang diterbitkan oleh [Will Schroeder](https://twitter.com/harmj0y) & [Lee Christensen](https://twitter.com/tifkin_) untuk memahami materi ini lebih lanjut.
{: .prompt-tip }

## 0x1 - Table of Contents

1. [Introduction to Active Directory Certificate Services (AD CS)](/posts/intro-to-ad-cs/)
2. [Misconfigured Certificate Template Exploit](/posts/misconfigured-certificate-template-exploit/)
3. [NTLM Relaying to ADCS HTTP Endpoints](/posts/ntlm-relaying-to-ad-cs-http/)
4. [User & Computer Persistence](/posts/ad-cs-user-and-computer-persistence/)
