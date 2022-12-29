---
title: Find The Origin IP Behind WAF via IP Ranges/CIDRs
date: 2022-02-16 17:20:00 +0800
categories: [Web Pentest, Bypass WAF]
tags: [web, waf]     # TAG names should always be lowercase
author: nairpaa
---

Salah satu cara untuk mem-*bypass* WAF adalah dengan cara mengakses aplikasi web target melalui IP aslinya. 

Umumnya perusahaan membeli banyak IP sekaligus (per-*subnet*) untuk digunakan oleh layanan-layanannya. Dan biasanya tidak semua layanan menggunakan WAF (contoh: server *development*). 

Untuk mencari *range* IP milik target, hal pertama yang akan kita lakukan adalah mencari sub-domain target. Setelah itu, kita kumpulkan IP yang digunakan oleh sub-domain target. Hal ini bisa dilakukan dengan mudah dilakukan menggunakan tool seperti [aquatune](https://github.com/michenriksen/aquatone).

Selanjutnya, dengan menggunakan tool seperti nmap kita akan mencari port web pada *range* IP yang telah kita temukan sebelumnya. Setelah mendapatkan list IP target yang menjalankan layanan web, kita akan mengkases virtual host domain target melalui list IP tersebut.

## Tahap eksploitasi

### Step 1: Deteksi Jenis WAF

Untuk mendeteksi apakah target menggunakan WAF dan jenis WAF yang digunakan, kita bisa menggunakan tool [wafw00f](https://github.com/EnableSecurity/wafw00f).

```bash
➜ wafw00f -l
➜ wafw00f https://www.example.org
```

Atau kita juga bisa melihat informasi dari IP yang digunakan oleh target.
```bash
➜ host www.example.co.id
```

### Step 2: Cari subnet IP perusahaan target melalui sub-domain

Lakukan penyisiran terhadap seluruh sub-domain untuk menemukan subnet jaringan yang dimiliki oleh perusahaan target.

```bash
➜ aquatone-discover --domain example.co.id
```

### Step 3: Konfirmasi kepemilikan subnet

Jika terdapat beberapa subnet, lakukan pengecekan kepemilikan subnet.

```bash
➜ whois 10.20.xx.0 | grep netname
➜ whois 10.20.yy.0 | grep netname
```

### Step 4: Scan port web pada subnet target

Selanjutnya lakukan *scanning* port pada subnet target dan cari IP yang menjalankan server web.

```bash
➜ nmap -p 443 -T4 -vv 10.20.xx.0/24
➜ nmap -p 443 -T4 -vv 10.20.yy.0/24
```

### Step 5: Cek virtual host server web

Setelah mendapatkan IP yang memiliki server web, akses server web tersebut dan bandingkan dengan web yang ter-*proxy* oleh WAF.

```bash
➜ curl -H "Host: www.example.co.id" 10.20.xx.xx
```

### Step 6: Tambahkan IP asli ke host file

Jika ada IP yang sesuai, tambahkan IP tersebut ke file **/etc/hosts**.


--- 

## Referensi

- [Finding The Origin IP Behind CDNs](https://infosecwriteups.com/finding-the-origin-ip-behind-cdns-37cd18d5275)
