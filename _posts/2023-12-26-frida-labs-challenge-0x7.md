---
title: '[Frida Labs] 07 - Hooking the Constructor'
date: 2023-12-26 16:42:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara meng-*hook* *constructor* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x7.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x7/Challenge%200x7.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - Challenge 0x7

Tantangan ini mirip dengan [challenge 0x5](/posts/frida-labs-challenge-0x5/), tetapi memiliki sedikit perbedaan. Di sini, kita akan mempelajari bagaimana cara meng-*hook* *constructor*.

Mari kita instal aplikasinya.

![Aplikasi Challenge 0x7](/assets/img/posts/frida-labs-challenge-0x7/1.png)
_Aplikasi Challenge 0x7_

Hanya terdapat teks "**Hello World!**" . Mari kita dekompilasi menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0x7/2.png)
_Class MainActivity_

Setelah diamati, polanya mirip dengan tantangan kita [sebelumnya](/posts/frida-labs-challenge-0x6/), tetapi kali ini nama metodenya adalah `flag`. 

Mari kita lihat *class* `Checker`.

![Class Checker](/assets/img/posts/frida-labs-challenge-0x7/3.png)
_Class Checker_

Terdapat sebuah *constructor* yang mengambil dua nilai integer. Nilai ini dimasukkan ke dalam variabel lokal `num1` dan `num2`. Untuk mendapatkan *flag*, nilai `num1` dan `num2` harus lebih tinggi dari 512.

```java
public void flag(Checker A) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, IllegalBlockSizeException, BadPaddingException {
	if (A.num1 > 512 && 512 < A.num2) {
	
	.........
	.........
	.........
	
	}
}
```

Salah satu cara untuk menyelesaikan tantangan ini adalah dengan membuat *instance* untuk *class* `Checker` dan mengubah nilai variabelnya. Setelah itu, kita bisa gunakan *instance* dari `MainActivity` untuk memangil fungsi `flag`.

Karena yang kita hadapi adalah *constructor*, pada saat menggunakan operator `$new`, kita perlu memasukkan argumen di *brackets* untuk *constructor*-nya.

```javascript
var checker_obj  = checker.$new(600,600);  //Class object
```

Jadi, kita bisa dengan mudah menyelesaikan tantangan ini dengan *script* berikut:

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x7.MainActivity', {
      onMatch: function(instance) { 
        console.log("Instance found");

        var checker = Java.use("com.ad2001.frida0x7.Checker");
        var checker_obj  = checker.$new(600,600); //Class object
        instance.flag(checker_obj) 
      },
      onComplete: function() {}
    });
});
```

![Mendapatkan Flag](/assets/img/posts/frida-labs-challenge-0x7/4.png)
_Mendapatkan Flag_

Kita berhasil dengan mudah mendapatkan *flag*, tetapi mari kita coba menggunakan cara *hooking constructor*.

## 0x3 - Hooking the constructor

Sangat mudah untuk meng-*hook constructor*. Ini hampir mirip dengan meng-*hook* sebuah metode. Berikut adalah *template script*-nya:

```javascript
Java.perform(function (){
 
    var <class_reference> = Java.use("<package_name>.<class>");
    <class_reference>.$init.implementation = function(<args>){

        /*
        
        */
        
    }

})
```

Seperti yang kita lihat, untuk meng-*hook constructor* kita menggunakan kata kunci `$init`.

Mari kita coba untuk meng-*hook* *constructor* `Checker`. 

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x7.Checker");
    a.$init.implementation = function(<args>){

        /*
        
        */
        
    }

})
```

Sekarang, kita panggil *constructor* asli dengan nilai argumen yang lebih tinggi dari 512, sehingga tersimpan di `num1` dan `num2`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x7.Checker");
    a.$init.implementation = function(param){

        this.$init(600,600);
        
    }

})
```

Mari kita jalankan *script* ini. Seperti yang kita ketahui, *constructor* akan dipanggil ketika sebuah *instance* dibuat. Di aplikasi ini, *instance* dari *class* `Checker` sudah dibuat di metode `onCreate`. Oleh karena itu, kita harus memuat *script* ini ketika *runtime* menggunakan opsi `-l`.


```bash
➜ frida -U -f com.ad2001.frida0x7 -l script.js
```
{: .nolineno }

![Mendapatkan Flag Melalui Hook Constructor](/assets/img/posts/frida-labs-challenge-0x7/5.png)
_Mendapatkan Flag Melalui Hook Constructor_

Dan kita berhasil mendapakan *flag*-nya. Jadi, ini lah cara kita meng-*hook* dan memanipulasi *constructor* menggunakan Frida.