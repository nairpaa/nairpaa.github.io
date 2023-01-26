---
title: 'Rooting Device Realme 3 Pro'
date: 2023-01-26 15:48:22 +0700
categories: ['Android Pentest', 'Rooting']
tags: [rooting, realme 3 pro, magisk, twrp]     # TAG names should always be lowercase
author: nairpaa
---

## Unlock Boot Loader

1. Pastikan bahwa USB *debugging* telah aktif.
2. Install `adb` dan `fastboot`.
```bash
➜ sudo apt install android-tools-adb android-tools-fastboot -y
```
3. *Downgrade* terlebih dahulu ke Android 10. Untuk itu, download [file .ozip](https://download.c.realme.com/flash/Rollbackpack/realme_3Pro/sign_RMX1851EX_11_OTA_1180_all_w236tl6xfsaL.ozip) yang dibutuhkan, lalu simpan pada salah satu direktori perangkat.
4. Masuk ke **Recover Mode** (matikan perangkat, lalu tekan tombol `power` dan `volume -` bersamaan).
5. Install file `.ozip` tersebut dengan memilih `English` -> `Install from storage` -> `From SD Card` -> lalu pilih `.ozip` yang sudah disimpan sebelumnya.
7. Tunggu hingga selesai.
8. Setelah itu download dan install [APK DeepTesting](https://drive.google.com/u/0/uc?id=1e0yUd5M7Q7CxW2NBcAOiwr1vp1IWNayr&export=download).
9. Buka aplikasi **DeepTesting**, pilih `Start applying` -> `Query verification status` -> `Start the in-depth test` untuk masuk ke dalam mode *fastboot*.
10. Jalankan perintah berikut untuk meng-*unlock bootloader*.
```bash
➜ fastboot devices
➜ fastboot flashing unlock
```

## Install TWRP + Root (Magisk)

1. Download [TWRP](https://twrp.me/oppo/realme3pro.html) pada PC.
2. Download [Magisk](https://github.com/topjohnwu/Magisk/releases), lalu simpan pada salah satu direktori perangkat. 
```bash
➜ adb devices
➜ adb reboot bootloader
```
3. Install TWRP.
```bash
➜ fastboot flash recovery <twrp.img>
```
4. Masuk ke TWRP dengan cara yang sama untuk masuk ke **Recovery Mode**.
5. Lalu install Magisk-nya.

---

Referensi:
- https://www.youtube.com/watch?v=vh2hK3RjV2w
- https://www.youtube.com/watch?v=t5Jb2QUCT9E