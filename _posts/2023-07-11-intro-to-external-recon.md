---
title: '01 - Introduction to External Reconnaissance'
date: 2023-07-11 21:06:22 +0700
categories: ['Red Team', 'External Reconnaissance']
tags: [red team, what-is, fundamental, intro, osint]     # TAG names should always be lowercase
author: nairpaa
---

Jika [Red Teaming Engagement](/posts/red-team-engagement/) tidak dimulai melalui metodologi "*assume branch*" dan kita perlu mendapatkan *initial entry* ke dalam jaringan target, proses *external reconnaissance* akan diperlukan.

Ada dua aspek utama dalam *recon*, yaitu:
1. **Organisational**
2. **Technical**

Selama *organisational recon*, kita berfokus pada pengumpulan informasi tentang organisasi. Hal ini dapat mencakup orang-orang yang bekerja di organisasi tersebut (nama, pekerjaan dan keahlian), struktur organisasi, lokasi dan hubungan bisnis.

Sedangkan pada *technical recon*, kita mencari sistem yang dimiliki oleh organisasi seperti situs web publik, server email, *remote access solutions*, vendor dan produk yang digunakan, terutama yang bersifat defensif seperti web proxy, firewall, email gateway, antivirus, dll.

Kedua informasi ini dapat dilakukan secara pasif atau pun aktif.

*Recon* pastif bergantung pada sumber pihak ketiga seperti Google, LinkedIn, Shodan, dan media sosial, yang mana kita tidak secara aktif menyentuh bagian dari jaringan target.

Sedangkan *recon* aktif, kita secara langsung berkomunikasi dengan komponen-komponen yang milik target, seperti mengunjungi situs web target atau melakukan *scanning* pada server target.

*Recon* aktif pada dasarnya lebih berisiko, karena memberikan indikasi potensial pertama bagi organisasi bahwa mereka sedang diawasi.

Ketika melakukan *recon* aktif (digital), pertimbangkan untuk menggunakan layanan proxy atau VPN agar IP publik kita tidak terekspos.

## 0x1 - What is External Reconnaissance?

**External Reconnaissance** adalah proses mengumpulkan informasi tentang target dari perspektif eksternal, dengan tujuan untuk mencari titik-titik kerentanan yang dapat dieksploitasi. Ini biasanya dilakukan sebagai bagian dari tahap awal dalam serangan siber, meskipun juga dapat digunakan oleh organisasi yang mencoba memahami dan memitigasi risiko mereka sendiri.

## 0x2 - What is OSINT?

**OSINT** (*Open Source Intelligence*) adalah metode pengumpulan informasi yang memanfaatkan sumber daya publik yang tersedia secara bebas. OSINT bukan hanya tentang mencari kerentanan, tetapi juga dapat digunakan untuk memahami profil dan karakteristik target secara lebih luas.

Jadi, *External Reconnaissance* dapat memanfaatkan teknik OSINT dalam prosesnya. Sebagai contoh, seorang penyerang bisa menggunakan informasi yang ditemukan melalui pencarian Google (satu bentuk OSINT) untuk memahami lebih lanjut tentang target mereka. Namun, OSINT juga bisa digunakan dalam banyak konteks lainnya yang tidak berhubungan dengan External Reconnaissance.

## 0x2 - Table of Content

1. [Intro to External Reconnaissance](/posts/intro-to-external-recon/)
2. [OSINT Tools](/posts/osint-tools/)
3. [Find The Origin IP Behind WAF via IP Ranges/CIDRs](/posts/bypass-waf-via-ip-ranges/)