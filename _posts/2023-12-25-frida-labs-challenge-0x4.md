---
title: '[Frida Labs] 04 - Creating a Class Instance'
date: 2023-12-25 20:16:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara membuat *class instance* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x4.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x4/app-debug.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - Challenge 0x4

Mari kita jalankan aplikasinya.

![Aplikasi Challenge 0x4](/assets/img/posts/frida-labs-challenge-0x4/1.png)
_Aplikasi Challenge 0x4_

Sederhana, tidak ada apa-apa. Mari kita dekompilasi menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0x4/2.png)
_Class MainActivity_

Tidak ada apa-apa di `MainActivity`. Mari kita lihat *class* `Check`.

![Class Check](/assets/img/posts/frida-labs-challenge-0x4/3.png)
_Class Check_

Terdapat metode `get_flag()`, yang berisi operasi XOR yang akan mendekode teks `I]FKNtW@]JKPFA\\[NALJr` menggunakan kunci `15`. Meskipun kita bisa dengan mudah menyelesaikannya, seperti yang telah disebutkan sebelumnya, tujuan kita adalah untuk menjadi lebih mahir menggunakan Frida. 

Metode ini tidak pernah dipanggil oleh aplikasi. Selain itu, metode ini memiliki argumen `a`, di mana jika nilai dari argumen `a` adalah `1337`, maka *flag* akan didekode.

Jika diperhatikan, metode `get_flag()` ini merupakan *instance method*. Oleh karena itu, kita tidak bisa memanggil metode tersebut dengan cara yang sama seperti memanggil [*static method*](/posts/frida-labs-challenge-0x2/).

> Baca lebih lanjut tentang perbedaan metode *static* dan *instance* [di sini](https://stackoverflow.com/questions/11993077/difference-between-static-methods-and-instance-methods).
{: .prompt-tip }

## 0x3 - Calling the get_flag()

Mari kita lihat bagaimana cara memanggil metode `get_flag()`. Seperti yang kita ketahui, ini bukan *static method*, jadi yang pertama kita harus lakukan adalah membuat *instance* dari *class* tersebut. Dengan menggunakan *intance* tersebut, kita akan memanggil metodenya. Berikut adalah kode Java yang sudah disesuaikan:

```java
Check ch = new Check();
String flag = ch.get_flag(1337);
```

Fungsi ini akan mengembalikan sebuah *string*, jadi kita harus menyimpannya di variabel.

Berikut adalah *tempalte script* yang akan kita gunakan:

```javascript
Java.perform(function (){
 
    var <class_reference> = Java.use("<package_name>.<class>");
    var <class_instance> = <class_reference>.$new(); //Class Object
    <class_instance>.<method>();   //Calling the method

})
```

Di Frida, untuk membuat *instance* dari *class* Java, kita bisa menggunakan metode `$new()`. Ini adalah metode spesifik dari Frida yang memungkinkan kita untuk menginstansiasi objek dari *class* tertentu.

Mari kita sesuaikan kodenya.
- *Package*: `com.ad2001.frida0x4`
- *Class*: `Check `
- *Function*:  `get_flag` 

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x4.Check");

})
```

Mari kita buat *instance*-nya menggunakan metode `$new()`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x4.Check");
    var a_obj = a.$new(); //Class Object

})
```

Sekarang, kita bisa dengan mudah memanggil metode `get_flag()`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x4.Check");
    var a_obj = a.$new(); //Class Object
    var res = a_obj.get_flag(1337);   //Calling the method
    console.log("FLAG:" + res)

})
```

Mari kita jalankan *script* tersebut.

```bash
➜ frida -U -f com.ad2001.frida0x4
```
{: .nolineno }

![Mendapatkan Flag Dari Instance Method](/assets/img/posts/frida-labs-challenge-0x4/4.png)
_Mendapatkan Flag Dari Instance Method_

Yey.. Kita mendapatkan *flag*.