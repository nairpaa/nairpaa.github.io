---
title: '.DS_Store File Disclosure'
date: 2022-05-28 05:00:22 +0700
categories: [Web Hacking, Web App Testing]
tags: [web, pentest]     # TAG names should always be lowercase
author: nairpaa
---

File [`.DS_Store` (Desktop Services Store)](http://en.wikipedia.org/wiki/.DS_Store) adalah file yang dibuat Mac OS X untuk pengaturan foldernya sama seperti file `desktop.ini` pada Windows.

File `.DS_Store` berisi tentang informasi posisi icon dalam folder, background folder, dan informasi lainnya.

File ini biasanya dibuat ketika kita mengakses sebuah folder melalui Finder.

Pengguna Windows biasanya mendapati file ini ketika menerima file dari pengguna Mac OS X.

Dengan menggunakan *script tools*, kita dapat mem-*parsing* file `.DS_Store` dengan mudah dan mendapatkan informasi seputar direktori tersebut.

Jika file `.DS_Store` ditemukan pada server web, kita dapat memanfaatkannya untuk melakukan enumerasi file dan path yang tersedia pada direktori web tersebut.

## Eksploitasi

### Step 1: Temukan File .DS_Store dan download

```bash
# menemukan file .DS_Store
➜ dirsearch -r -u http://example.com
```

![Menemukan file .DS_Store](/assets/img/posts/ds-store-disclosure/2.png)

### Step 2: Parsing File .DS_Store

```bash
# clone script
➜ git clone https://github.com/lijiejie/ds_store_exp

# parse script for .DS_Store
➜ cd ds_store_exp
➜ python ds_store_exp.py http://example.com/.DS_Store
```


![Parsing file .DS_Store](/assets/img/posts/ds-store-disclosure/2.png)


## Referensi

- https://winpoin.com/winexplain-apa-itu-file-thumbs-db-desktop-ini-dan-ds_store/
- https://makemac.grid.id/read/21971829/menghapus-dan-mencegah-pembuatan-ds_store-di-mac
- https://github.com/lijiejie/ds_store_exp
- https://0xdf.gitlab.io/2020/06/08/endgame-poo.html#ds_store-enumeration