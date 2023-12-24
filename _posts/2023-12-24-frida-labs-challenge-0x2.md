---
title: '[Frida Labs] 02 - Calling a Static Method'
date: 2023-12-24 11:03:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara memanggil *static method* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x2.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x2/Challenge%200x2.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - Challenge 0x2

Mari kita jalankan aplikasinya.

![Aplikasi Challenge 0x2](/assets/img/posts/frida-labs-challenge-0x2/1.png)
_Aplikasi Challenge 0x2_

Aplikasi ini cukup sederhana, hanya terdiri dari sebuah `TextView`, tanpa adanya tombol atau elemen tambahan. Teks yang tampil di `TextView` adalah '**HOOK ME!**', ini sesuai dengan apa yang akan kita lakukan. 

Mari kita gunakan `JADX` untuk melakukan *reverse engineering* aplikasi.

![Reverse Engineering APK](/assets/img/posts/frida-labs-challenge-0x2/2.png)
_Reverse Engineering APK_

Seperti yang kita lihat, ini adalah aplikasi sederhana. Satu-satunya hal yang dapat dilakukan oleh aplikasi adalah menampilkan `TextView`. Dan bisa kita lihat, metode `get_flag()` berfungsi untuk mendekripsi *flag* dan menampilkannya di `TextView`. Namun, metode ini tidak pernah dipanggil.  

Metode `get_flag()` memiliki kondisi `a == 4919`, di mana `a` adalah argumen yang ada pada metode `get_flag()`.

Jadi untuk mendapatkan *flag* menggunakan Frida, kita cukup memanggil metode `get_flag()` dengan memberikan argumen berupa integer `4919`.

Mari kita mulai dengan mencari *identifier* dari *package* aplikasi.

```bash
➜ frida-ps -Uai
```

![Mencari Nama Package Aplikasi Melalui Frida](/assets/img/posts/frida-labs-challenge-0x2/3.png)
_Mencari Nama Package Aplikasi Melalui Frida_

*Identifier* dari aplikasi yang kita tuju adalah `com.ad2001.frida0x2`. Kita juga bisa menemukannya di `JADX`.

![Mencari Nama Package Aplikasi Melalui Reverse Engineering](/assets/img/posts/frida-labs-challenge-0x2/4.png)
_Mencari Nama Package Aplikasi Melalui Reverse Engineering_

Mari kita mulai menyiapkan *template script* Frida untuk memanggil *static method*.

```javascript
Java.perform(function (){
 
    var <class_reference> = Java.use("<package_name>.<class>");
    <class_reference>.<static_method>();

})
```

Pertama, kita harus menentukan *class reference*-nya, yaitu `MainActivity`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x2.MainActivity");

})
```

Selanjutnya, kita panggil metode `get_flag()` dengan argumen `4919` untuk menampilkan *flag*.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x2.MainActivity");
    a.get_flag(4919);  //method name

})
```

Mari kita jalankan *script* tersebut.

```bash
frida -U -f com.ad2001.frida0x2
```

![Mendapatkan Flag Dengan Memanggil Static Method](/assets/img/posts/frida-labs-challenge-0x2/5.png)
_Mendapatkan Flag Dengan Memanggil Static Method_

Saat *script* tersebut dijalankan, *flag* akan muncul di aplikasi.

Itulah cara memanggil *static method* di Frida, semoga bermanfaat.