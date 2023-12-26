---
title: '[Frida Labs] 05 - Invoking Methods on an Existing Instance'
date: 2023-12-26 11:24:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara memanggil metode pada *instance* yang sudah ada menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x5.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x5/Challenge%200x5.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - Challenge 0x5

Mari kita jalankan aplikasinya.

![Aplikasi Challenge 0x5](/assets/img/posts/frida-labs-challenge-0x5/1.png)
_Aplikasi Challenge 0x5_

Seperti tantangan sebelumnya, tidak ada apa pun di UI. Mari kita dekompilasi menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0x5/2.png)
_Class MainActivity_

Pada `MainActivity`, terdapat metode `flag` yang tidak pernah dipanggil oleh aplikasi. Metode ini mendekripsi *flag* dan menampilkannya di `TextView`. Selain itu, perhatikan bahwa kita harus memberikan nilai `1337` sebagai argumen untuk melewati pemeriksaan `if`. 

Situasi ini mirip dengan tantangan [sebelumnya](/posts/frida-labs-challenge-0x4/), tetapi kali ini metode berada di `MainActivity`. Mungkin kita bertanya, apa bedanya? Kenapa kita tidak membuat *instance* dari `MainActivity` dan memanggil metode `flag`? Mari kita coba.

- *Package*: `com.ad2001.frida0x5`
- *Class*: `MainActivity `
- *Function*:  `flag` 

Mari kita tinjau *tempalte script*-nya kembali.

```javascript
Java.perform(function (){
 
    var <class_reference>= Java.use("<package_name>.<class>");
    var <class_instance> = <class_reference>.$new(); //Class Object
    <class_instance>.<method>();   //Calling the method

})
```

```javascript
Java.perform(function (){
 
    var a= Java.use("com.ad2001.frida0x5.MainActivity");
    var main_act = a.$new(); //Class Object
    main_act.flag(1337);   //Calling the method

})
```

Jalankan *script* Frida-nya.

```bash
➜ frida -U -f com.ad2001.frida0x5 
```
{: .nolineno }

![Tidak Bisa Membuat Instance Dari Class MainActivity](/assets/img/posts/frida-labs-challenge-0x5/3.png)
_Tidak Bisa Membuat Instance Dari Class MainActivity_

Lihat, *script*-nya *crash*. Kenapa ini bisa terjadi?

Membuat *instance* dari `MainActivity` atau komponen Android lainnya secara langsung menggunakan Frida bisa menjadi rumit karena aturan *lifecycle* dan *threading* Android. Komponen-komponen ini memerlukan konteks aplikasi dan *thread* tertentu untuk berfungsi dengan benar.

Pada Frida, kita mungkin kekurangan konteks ini dan kesulitan mengatur *lifecycle* Android secara keseluruhan. Oleh karena itu, membuat *instance* untuk `MainActivity` secara langsung tidak disarankan.

Jadi, apa solusinya?

Ketika aplikasi Android dijalankan, sistem akan membuat *instance* dari `MainActivity` (atau *launcher activity* yang ditentukan di file **AndroidManifest.xml**). Pembuatan *instance* `MainActivity` ini adalah bagian dari *lifecycle* aplikasi Android. Jadi kita bisa menggunakan Frida untuk mendapatkan *instance* `MainActivity` tersebut, lalu memanggil metode `flag()` untuk mendapatkan *flag*.

## 0x3 - Invoking methods on an existing instance

Kita bisa dengan mudah untuk memanggil metode pada *instance* yang sudah ada menggunakan Frida. Setidaknya, kita akan menggunakan dua API, yaitu:
- `Java.performNow` : digunakan untuk menjalankan kode dalam *runtime* Java.
- `Java.choose`:  digunakan untuk menemukan dan berinteraksi dengan *class instance* Java yang sudah ada dalam *runtime*. Ini memungkinkan kita memanipulasi *instance* tersebut.

Berikut adalah *template script* yang akan kita gunakan:

```javascript
Java.performNow(function(){
    Java.choose('<Package>.<class_Name>', {
      onMatch: function(instance) {
        // TO DO//
      },
      onComplete: function() {}
    });
});
```

Terdapat dua *callback* di sini:
- **onMatch**
	- Berfungsi untuk melakukan aksi pada setiap *instance* yang ditemukan.
	- `function(instance) {}`, parameter `instance` mewakili setiap *instance* yang cocok dari *class* target. Kita bisa menggunakan nama lain yang kita inginkan.
- **onComplete**
	- Berfungsi untuk melakukan aksi setelah pencarian selesai dan bersifat opsional.

Sekarang kita sudah mengetahui kegunaan dari API `Java.choose`. Mari kita sesuaikan *script*-nya.

- *Package*: `com.ad2001.frida0x5`
- *Class*: `MainActivity `
- *Function*:  `flag` 

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x5.MainActivity', {
      onMatch: function(instance) {
        // TO DO//
      },
      onComplete: function() {}
    });
});
```

Kita masukkan *statement* `console.log` untuk menampilkan bahwa kita telah berhasil menemukan *instance* `MainActivity`.

Kita akan membuat `onComplete` kosong karena kita tidak akan melakukan tindakan apa pun setelah pencarian selesai.

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x5.MainActivity', {
      onMatch: function(instance) {
        console.log("Instance found");
      },
      onComplete: function() {}
    });
});
```

Mari kita jalankan *script*-nya.

```bash
➜ frida -U -f com.ad2001.frida0x5
```
{: .nolineno }

> **Perhatian!**
> ![](https://github.com/DERE-ad2001/Frida-Labs/blob/main/Frida%200x5/Solution/images/3.png?raw=true)
> _Kesalahan Saat Menjalankan Script_
> Jika kamu mengalami *crash* seperti pada gambar di atas, itu mungkin karena kesalahan sistem yang kadang terjadi, bukan karena kesalahan *script*. Saran untuk mengatasinya adalah dengan mencoba pada perangkat Android asli dan pastikan kamu menggunakan versi Frida yang terbaru.
{: .prompt-warning }

![Instance Berhasil Ditemukan](/assets/img/posts/frida-labs-challenge-0x5/4.png)
_Instance Berhasil Ditemukan_

Terlihat, *instance* `MainActivity` ditemukan. Sekarang kita bisa memanggil metode `flag` dan memberikan nilai `1337`. Kita bisa menggunakan format `.<function_name>()` seperti yang kita lakukan pada tantangan [sebelumnya](/posts/frida-labs-challenge-0x4/).

```javascript
Java.performNow(function(){
    Java.choose('com.ad2001.frida0x5.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        console.log("Instance found");
        instance.flag(1337); //Calling the function
      },
      onComplete: function() {}
    });
});
```

Mari kita jalankan *script*-nya.

![Berhasil Mendapakan Flag](/assets/img/posts/frida-labs-challenge-0x5/5.png)
_Berhasil Mendapakan Flag_

Setelah *script* tersebut berjalan, aplikasi akan menampilkan *flag* di `TextView`. 
