---
title: '[Frida Labs] 06 - Invoking a Method with an Object Argument'
date: 2023-12-26 14:55:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara memanggil metode dengan argumen objek menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x6.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x6/Challenge%200x6.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - Challenge 0x6

Mari kita jalankan aplikasinya.

![Aplikasi Challenge 0x6](/assets/img/posts/frida-labs-challenge-0x6/1.png)
_Aplikasi Challenge 0x6_

Sekilas, tidak ada yang menarik dari aplikasi ini. Mari kita dekompilasi menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0x6/2.png)
_Class MainActivity_

Ini mirip seperti tantangan [sebelumnya](/posts/frida-labs-challenge-0x4/). Dalam kasus ini, metode `get_flag()` tidak pernah dipanggil oleh aplikasi.

Jika kita perhatikan, metode `get_flag()` memiliki satu argumen, yang mana ini adalah *instance* dari *class* `Checker`. Argumen ini diberi nama `A`.

```java
public void get_flag(Checker A) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException {
    // Method body
}
```

Di dalam metode tersebut, terdapat pemeriksaan apakah `A.num1` sama dengan `1234` dan `A.num2` sama dengan `4321`. Jika kondisi ini terpenuhi, aplikasi akan mendekripsi *flag* dan menampilkannya di `TextView`.

Mari kita periksa *class* `Checker`.

![Class Checker](/assets/img/posts/frida-labs-challenge-0x6/3.png)
_Class Checker_

Di dalam *class* `Checker` terdapat dua variabel, yaitu:
- `num1`
- `num2`

`num1` harus bernilai `1234` dan `num2` harus bernilai `4321` untuk kita bisa mendapatkan *flag*.

## 0x3 - Solution

Sangat mudah untuk menyelesaikan tantangan ini, karena kita sudah melakukan hal serupa [sebelumnya](/posts/frida-labs-challenge-0x3/). Hanya saja kali ini terdapat perbedaan, yaitu argumen metode `get_flag` adalah objek dari *class* `Checker`.

Berikut adalah langkah-langkah yang akan kita lakukan:
1. Membuat *instance* untuk *class* `Checker`.
2. Menyetel `num1` menjadi `1234` dan `num2` menjadi `4321`.
3. Dapatkan *instance* dari `MainActiviy`.
4. Panggil metode `get_flag` menggunakan *insance* sebagai argumen.

Mari kita buat *script*-nya.

Pertama, kita buat *insance* untuk *class* `Checker`.

```javascript
var checker = Java.use("com.ad2001.frida0x6.Checker");
var checker_obj  = checker.$new();  //Class object
```

Lalu, atur nilai dari variabel `num1` dan `num2`.

```java
if (1234 == A.num1 && 4321 == A.num2) {
	........
	........
	........
}
```

```javascript
checker_obj.num1.value = 1234;
checker_obj.num2.value = 4321;
```

Sekarang, dapatkan *instance* `MainActivity`. Untuk melakukan ini gunanakan API `Java.performNow` dan `Java.choose` seperti di tantangan [sebelumnya](/posts/frida-labs-challenge-0x5/).

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x6.MainActivity', {
      onMatch: function(instance) { 
        console.log("Instance found");
      },
      onComplete: function() {}
    });
});
```

Gabungkan *script*-nya dengan *instance* dari *class* `Checker`.

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x6.MainActivity', {
      onMatch: function(instance) { 
        console.log("Instance found");

        var checker = Java.use("com.ad2001.frida0x6.Checker");
        var checker_obj  = checker.$new();  //Class object
        checker_obj.num1.value = 1234; //num1
        checker_obj.num2.value = 4321; //num2
      },
      onComplete: function() {}
    });
});
```

Terakhir, kita tinggal memanggil metode `get_flag` dengan memasukan *instance* dari *class* `Checker` sebagai argumennya.

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x6.MainActivity', {
      onMatch: function(instance) { 
        console.log("Instance found");

        var checker = Java.use("com.ad2001.frida0x6.Checker");
        var checker_obj  = checker.$new();  //Class object
        checker_obj.num1.value = 1234; //num1
        checker_obj.num2.value = 4321; //num2
        instance.get_flag(checker_obj) //invoking the get_flag method
      },
      onComplete: function() {}
    });
});
```

Jalankan _script_ Frida-nya.

```bash
➜ frida -U -f com.ad2001.frida0x6
```
{: .nolineno }

![Berhasil Mendapatkan Flag](/assets/img/posts/frida-labs-challenge-0x6/4.png)
_Berhasil Mendapatkan Flag_

Yey.. Kita berhasil mendapatkan _flag_.
