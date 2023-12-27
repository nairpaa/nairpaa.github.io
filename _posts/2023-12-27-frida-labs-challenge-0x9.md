---
title: '[Frida Labs] 09 - Changing the return value of a native function'
date: 2023-12-27 23:18:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara mengubah *return value* dari fungsi *native* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x9.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x9/Challenge%200x9.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.
- *Basics of x64/ARM64 assembly and reversing*.

## 0x2 - Challenge 0x9

Mari kita mulai dengan menginstal aplikasi.

![Aplikasi Challenge 0x9](/assets/img/posts/frida-labs-challenge-0x9/1.png)
_Aplikasi Challenge 0x9_

Setelah instalasi, kita dihadapkan dengan satu tombol. Mari kita coba mengekliknya.

![Klik Tombol](/assets/img/posts/frida-labs-challenge-0x9/2.png)
_Klik Tombol_

Setelah mengklik, muncul pesan "**Try again**". Kita akan menggunakan `JADX` untuk dekompilasi dan menganalisis aplikasi lebih lanjut.

![Class MainActity](/assets/img/posts/frida-labs-challenge-0x9/3.png)
_Class MainActity_

Kita menemukan deklarasi fungsi *native* bernama `check_flag`, yang terdefinisi dalam *library* `a0x9`. Fungsi ini tidak mengambil argumen dan mengembalikan integer. Ketika tombol diklik, aplikasi membandingkan nilai yang dikembalikan dari `check_flag` dengan `1337`. Jika nilainya sama, aplikasi mendekripsi *flag* dan menampilkannya, jika tidak, pesan "Try again" muncul. Jadi, untuk mendapatkan *flag*, kita perlu membuat `check_flag` mengembalikan `1337`.

Tanpa berlama-lama lagi, mari kita analisis *library* `a0x9` menggunakan Ghidra.

![List Library](/assets/img/posts/frida-labs-challenge-0x9/4.png)
_List Library_

Setelah memuat *library* di Ghidra, kita menemukan bahwa fungsi `check_flag` hanya mengembalikan `1`. Untuk mendapatkan *flag*, kita bisa meng-*hook* metode ini dan membuatnya mengembalikan `1337`.

![Analisis Ghidra](/assets/img/posts/frida-labs-challenge-0x9/5.png)
_List Library_

Nama fungsi dalam *native space* adalah `Java_com_ad2001_a0x9_MainActivity_check_1flag`.

Sekarang, mari kita mulai menulis *script* Frida kita:

```javascript
Interceptor.attach(targetAddress, {
    onEnter: function (args) {
        console.log('Entering ' + functionName);
        // Modify or log arguments if needed
    },
    onLeave: function (retval) {
        console.log('Leaving ' + functionName);
        // Modify or log return value if needed
    }
});
```

Kita akan mulai dengan menemukan alamat untuk `check_flag` menggunakan `Module.enumerateExports`.

![Mencari Address check_flag](/assets/img/posts/frida-labs-challenge-0x9/6.png)
_Mencari Address check_flag_

```javascript
var check_flag = Module.enumerateExports("liba0x9.so")[0]['address']
Interceptor.attach(check_flag, {
    onEnter: function (args) {
      
    },
    onLeave: function (retval) {

    }
});
```

Langkah selanjutnya adalah mengubah nilai kembalian. Kita akan mengosongkan blok `onEnter` dan fokus pada `onLeave` untuk mengubah nilai kembalian. `retval` adalah nilai kembalian asli yang dapat kita ubah dengan `retval.replace()`.

Mari kita lakukan perubahan tersebut:

```javascript
var check_flag = Module.enumerateExports("liba0x9.so")[0]['address']
Interceptor.attach(check_flag, {
    onEnter: function (args) {
        
    },
    onLeave: function (retval) {
        console.log("Original return value :" + retval);
	    retval.replace(1337)  //changing the return value to 1337.
    }
});
```

Sekarang, mari kita jalankan *script* dan klik tombol untuk melihat apakah kita berhasil mendapatkan *flag* kita.

![Mendapatkan Flag](/assets/img/posts/frida-labs-challenge-0x9/7.png)
_Mendapatkan Flag_

Dan dengan mudah kita mendapatkan *flag*.