---
title: '[Kerberos] 01 - Introduction'
date: 2023-08-01 21:39:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, introduction, what is, active directory hacking, windows hacking, post compromise attack, kerberos]     # TAG names should always be lowercase
author: nairpaa
---


Kerberos adalah topik yang seru dan terdapat beberapa penyalahgunaan yang terkenal di dalam lingkungan Active Directory.

## 0x1 - What is Kerberos?

Kerberos adalah protokol autentikasi yang digunakan dalam banyak sistem operasi, termasuk lingkungan Active Directory dari Microsoft. Protokol ini membantu dalam memvalidasi identitas pengguna atau layanan di jaringan, dan didesain untuk memberikan metode autentikasi yang kuat dan aman.

## 0x2 - How does Kerberos Work?

Berikut ini adalah gambaran singkat tentang cara kerja Kerberos:

![Cara Kerja Kerberos](https://www.qomplx.com/content/images/2019/07/kerberos-diagram.png)

Penjelasan:

1. **Client request TGT (AS-REQ)**: Ketika pengguna pertama kali *login*, klien (biasanya komputer pengguna) mengirim permintaan *Authentication Service Request* (AS-REQ) ke *Key Distribution Center* (KDC). Permintaan ini mencakup nama pengguna dan domain mereka. Tujuan dari permintaan ini adalah untuk mendapatkan *Ticket Granting Ticket (TGT)*, yang dapat digunakan nanti untuk meminta tiket akses ke *service* tertentu.
2. **KDC returns TGT (AS-REP)**: KDC memverifikasi permintaan dengan mencocokkan nama pengguna dengan password yang disimpan dalam Active Directory. Jika ini cocok, KDC menghasilkan TGT dan mengirimkannya kembali ke klien dalam pesan *Authentication Service Reply (AS-REP)*. TGT ini dienkripsi dengan kunci rahasia KDC (biasanya menggunakan akun `krbtgt`), sehingga tidak dapat dibaca atau dimodifikasi oleh klien.
3. **Client requests TGS for service (TGS-REQ)**: Ketika klien ingin mengakses suatu *service* (misalnya *file share*), klien mencari *Service Principal Name* (SPN) dari *service* tersebut dan mengirim permintaan *Ticket Granting Service Request* (TGS-REQ) ke KDC. Permintaan ini mencakup TGT yang diterima sebelumnya dan SPN dari layanan yang ingin diakses.
4. **KDC returns TGS (TGS-REP)**: KDC memverifikasi TGT dan jika valid, menghasilkan *Ticket Granting Service* (TGS) untuk *service* yang diminta. TGS ini kemudian dikirimkan kembali ke klien dalam pesan *Ticket Granting Service Reply (TGS-REP)*. TGS dienkripsi dengan kunci rahasia *service* yang diminta, sehingga hanya *service* itu sendiri yang bisa membukanya.
5. **Client presents TGS to service**: Klien kemudian mengirim TGS yang diterima kepada *service* yang diminta. *Service* ini memverifikasi TGS dan jika valid, mengizinkan akses.
6. **Service grants access**: Setelah verifikasi, *service* memberikan akses ke pengguna berdasarkan hak dan izin yang ditetapkan.

## 0x3 - Table of Contents

1. [Introduction to Kerberos](/posts/intro-to-kerberos/)
2. [Kerberoasting](/posts/kerberoasting/) 
3. [AS-REP Roasting](/posts/as-rep-roasting/)
3. Unconstrained Delegation 
4. Constrained Deletagation 
5. Alternate Service Name 
6. S4U2Self Abuse 
7. Resource-Based Constrained Delegation 
8. Shadow Credentials

---

## 0x4 - References

- https://learn.microsoft.com/id-id/windows-server/security/kerberos/kerberos-authentication-overview
- https://www.simplilearn.com/what-is-kerberos-article
- https://www.qomplx.com/about-kerberos/