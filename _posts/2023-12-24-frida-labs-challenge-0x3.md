---
title: '[Frida Labs] 03 - Changing The Value of a Variable'
date: 2023-12-24 20:34:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam bagian ini, kita akan mempelajari cara mengubah nilai variabel menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi '**Challenge 0x3.apk**', yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x3/Challenge%200x3.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - Challenge 0x3

Mari kita jalankan aplikasinya.

![Aplikasi Challenge 0x3](/assets/img/posts/frida-labs-challenge-0x3/1.png)
_Aplikasi Challenge 0x3_


Aplikasi memiki sebuah tombol dan `TextView`. `TextView` tersebut memberikan *hint* tentang *flag*. Tidak ada `EditText` untuk meng-input-kan apapun. Jika kita mengklik tombol tersebut, maka akan muncul pesan "**TRY AGAIN**". 

Mari kita analisis aplikasinya menggunakan `JADX`.

![Reverse Engineering APK](/assets/img/posts/frida-labs-challenge-0x3/2.png)
_Reverse Engineering APK_

Kali ini kita memiliki *class* tambahan, yaitu `Checker`.

![Class Checker](/assets/img/posts/frida-labs-challenge-0x3/3.png)
_Class Checker_

Pada *class* ini, terdapat variable integer statis bernama `code` yang diberi nilai `0`. Selain itu, ada *static method* bernama `increase()`. Metode ini akan menambahkan 2 nilai ke variabel `code` ketika dipanggil. Metode ini tidak pernah dipanggil oleh aplikasi, jadi nilai dari variable `code` tidak akan berubah. 

Mari kita perhatikan kode di `MainActivity`.

```java
btn.setOnClickListener(new View.OnClickListener() { // from class: com.ad2001.frida0x3.MainActivity.1
            @Override // android.view.View.OnClickListener
            public void onClick(View v) {
                if (Checker.code == 512) {
                    byte[] bArr = new byte[0];
                    ...
                    ...
                }                
```

Ketika tombol diklik, aplikasi akan melakukan validasi apakah variabel `code` pada *class* `Checker` sama dengan `512`. Jika kondisi ini terpenuhi, aplikasi akan memberikan pesan "**You won**", mendekripsi *flag*, dan menampilkannya di `TextView`. Jika tidak, ia akan menampilkan pesan "**TRY AGAIN**".

Terdapat dua cara utama yang bisa kita lakukan, yaitu:
- Mengubah nilai variabel `code` menjadi `512`.
- Memanggil metode `increase()` sebanyak 256 kali.

## 0x3 - Changing the value of code variable

Untuk mendapatkan *flag*, kita perlu mengatur nilai dari variable `code` ke `512`. Kita dapat melakukannya dengan mudah menggunakan Frida.

Berikut adalah *template script* untuk mengubah nilai variable menggunakan Frida:

```javascript
Java.perform(function (){
 
    var <class_reference> = Java.use("<package_name>.<class>");
    <class_reference>.<variable>.value = <value>;

})
```

Mari kita sesuaikan kodenya.

- *Package*: `com.ad2001.frida0x3`
- *Class*: `Checker`
- *Variable*: `code`

```java
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x3.Checker");
    a.code.value = 512;

})
```

Mari kita jalankan *script*-nya.

```
frida -U -f com.ad2001.frida0x3
```

![Mengubah Nilai Variable Menggunakan Frida](/assets/img/posts/frida-labs-challenge-0x3/4.png)
_Mengubah Nilai Variable Menggunakan Frida_

Sebelum *script*-nya dijalankan, jika kita mengklik tombolnya, maka akan muncul pesan "**TRY AGAIN**". Sekarang, setelah setelah *script*-nya dijalankan, aplikasi akan menampilkan *flag*. 

Bagaimana, mudah bukan? Mari kita coba cara yang kedua.

## 0x4 - Calling the increase() method

Pendekatalan lainnya adalah dengan memanggil metode `increase()` sebanyak 256 kali. Setiap pemanggilan meode ini akan menambah nilai dari variable `code` sebesar 2. 

[Sebelumnya](/posts/frida-labs-challenge-0x2/) kita telah mempelajari cara untuk memanggil *static method* di Frida, jadi untuk memanggil metode `increase()` berkali-kali, kita bisa menggunakan *loop*.

Mari kita buat *script*-nya.

- *Package*: `com.ad2001.frida0x3`
- *Class*: `Checker`
- *Method*: `increase()`

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x3.Checker");

    for (var i = 1; i <= 256; i++) {
        console.log("Calling increase() method " + i + " times");
        a.increase();
    }

})
```

Tutup aplikasinya dan jalankan kembali menggunakan Frida.

```
 frida -U -f com.ad2001.frida0x3
```

Sekarang, jalankan *script*-nya.

![Xyz](/assets/img/posts/frida-labs-challenge-0x3/5.png)
_Xyz_

Bisa kita lihat, *script* tersebut akan memanggil metode `increase()` berulang kali dan nilai variable `code` akan menjadi `256`.  Ketika kita klik tombolnya, aplikasi akan menampilkan *flag* seperti yang kita harapkan.