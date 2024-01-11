---
title: '[FridaLab] All Challenges Walktrough'
date: 2024-01-10 17:13:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'fridalab']     # TAG names should always be lowercase
author: nairpaa
---

Setelah sebelumnya kita mempelajari [Frida](/posts/intro-to-android-app-hacking/#4-frida) menggunakan aplikasi [Frida-Labs](https://github.com/DERE-ad2001/Frida-Labs) karya [DERE-ad2001](https://github.com/DERE-ad2001/), pada kesempatan ini kita akan mengasah kemampuan Frida lebih lanjut dengan menyelesaikan soal CTF dari aplikasi [FridaLab](https://rossmarks.uk/blog/fridalab/) karya [Ross Marks](https://rossmarks.uk/about).

> Sebelum melihat _walkthrough_ ini, saya sangat menyarankan kamu untuk mencoba menyelesaikan soal tersebut terlebih dahulu menggunakan keterampilan yang telah kita pelajari sebelumnya.
{: .prompt-tip }

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.
- Familiar dengan [Frida](/posts/intro-to-android-app-hacking/#4-frida).

## 0x2 - FridaLab App

Mari kita unduh aplikasinya [di sini](https://rossmarks.uk/portfolio/FridaLab.apk), lalu instal dan jalankan.

![Aplikasi FridaLab](/assets/img/posts/fridalab-walkthrough/1.png)
_Aplikasi FridaLab_

Pada aplikasi ini terdapat 8 tantangan yang harus kita selesaikan dan terdapat tombol `CHECK`. Jika kita klik tombol tersebut, maka semua soal tantangan akan menjadi berwarna merah.

![Soal Tantangan Menjadi Warna Merah](/assets/img/posts/fridalab-walkthrough/2.png)
_Soal Tantangan Menjadi Warna Merah_

Jika kita analisis menggunakan `JADX`, ketika mengklik tombol `CHECK`, fungsi `changeColors()` dipanggil.

![Fungsi Tombol CHECK](/assets/img/posts/fridalab-walkthrough/3.png)
_Fungsi Tombol CHECK_

Fungsi ini akan mengubah warna soal menjadi hijau jika nilai dari setiap indeks array `completeArr` bernilai `1`, yang menandakan bahwa tantangan berhasil diselesaikan.

Urutan indeks array ini mencerminkan jawaban dari setiap tantangan. Indeks `0` untuk tantangan 1, indeks `1` untuk tantangan 2, dan seterusnya.

![Fungsi changeColors()](/assets/img/posts/fridalab-walkthrough/4.png)
_Fungsi changeColors()_

Untuk menyelesaikan tantangan-tantangan ini, kita bisa mengubah nilai array `completeArr` menjadi `1` untuk setiap indeksnya. Mari kita coba.

Pertama kita cari tahu dulu *identifier* dari aplikasi ini. Diketahui *identifier*-nya adalah `uk.rossmarks.fridalab`.

![Cek Identifier Aplikasi](/assets/img/posts/fridalab-walkthrough/5.png)
_Cek Identifier Aplikasi_

Jalankan aplikasinya menggunakan Frida.

```bash
➜ frida -U -f uk.rossmarks.fridalab
```
{: .nolineno }

Gunakan *script* berikut untuk mengubah nilai array `completeArr` menjadi `1` disetiap indeksnya.

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) {
        instance.completeArr.value = [1, 1, 1, 1, 1, 1, 1, 1];
      },
      onComplete: function() {}
    });
});
```

![Berhasil Menyelesaikan Tantangan Aplikasi](/assets/img/posts/fridalab-walkthrough/6.png)
_Berhasil Menyelesaikan Tantangan Aplikasi_

Hasilnya, semua tantangan berubah menjadi hijau kecuali tantangan nomor 5. Hal ini disebabkan karena fungsi `chall05()` yang dipanggil ketika tombol `CHECK` ditekan menggunakan argumen yang membuat nilai array menjadi `0`. 

```java
public void onClick(View view) {
    ...
	MainActivity.this.chall05("notfrida!");
	...
}

...
...

public void chall05(String str) {
	if (str.equals("frida")) {
		this.completeArr[4] = 1;
	} else {
		this.completeArr[4] = 0;
	}
}
```

Hahaha.. Meskipun kita berhasil menyelesaikan banyak tantangan dengan "curang", kita akan mencoba menyelesaikan setiap tantangan sesuai keinginan *Lab Maker*.

## 0x3 - Change class challenge_01 variable 'chall01' to: 1

Dengan membaca soal dan menganalisis kode Java-nya, kita bisa mengetahui bahwa untuk menyelesaikan tantangan 1 ini, kita perlu mengubah nilai dari variabel statis `chall1` pada *class* `challenge_01` menjadi `1`.

![Kode Tantangan 1](/assets/img/posts/fridalab-walkthrough/7.png)
_Kode Tantangan 1_

![Class Challenge_01](/assets/img/posts/fridalab-walkthrough/8.png)
_Class Challenge_01_

Untuk melakukan perubahan nilai pada variabel statis, kita bisa menggunakan konsep yang telah kita pelajari sebelumnya dalam [seri Frida-Labs Challenge](/posts/frida-labs-challenge-0x3/).

Berikut adalah *template script* yang akan kita gunakan:

```javascript
Java.perform(function (){
 
    var <class_reference> = Java.use("<package_name>.<class>");
    <class_reference>.<variable>.value = <value>;

})
```

Lakukan penyesuaian dengan mengubah nilai variabel `chall01` menjadi `1`.

```javascript
Java.perform(function (){
 
    var a = Java.use("uk.rossmarks.fridalab.challenge_01");
    a.chall01.value = 1;

})
```

Selanjutnya, jalankan *script*-nya dan tekan tombol `CHECK`. Terlihat bahwa kita berhasil menyelesaikan tantangan 1.

![Berhasil Menyelesaikan Tantangan 1](/assets/img/posts/fridalab-walkthrough/9.png)
_Berhasil Menyelesaikan Tantangan 1_

## 0x4 - Run chall02()

Untuk menyelesaikan tantangan 2, kita hanya perlu memanggil fungsi `chall02()`.

![Fungsi chall02()](/assets/img/posts/fridalab-walkthrough/10.png)
_Fungsi chall02()_

Untuk memanggil fungsi `chall02()` kita memerlukan akses ke [*class instance*](/posts/frida-labs-challenge-0x5/) `MainActivity`.

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

Mari kita sesuaikan dengan memanggil fungsi `chall02()`.

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        instance.chall02(); //Calling the function
      },
      onComplete: function() {}
    });
});
```

Jalankan *script* tersebut dan kita akan berhasil menyelesaikan tantangan 2.

![Berhasil Menyelesaikan Tantangan 2](/assets/img/posts/fridalab-walkthrough/11.png)
_Berhasil Menyelesaikan Tantangan 2_

## 0x5 - Make chall03() return true

Untuk menyelesaikan tantangan 3, kita perlu mengubah [nilai pengembalian](/posts/frida-labs-challenge-0x1/#a-hooking-the-get_random-method) dari fungsi `chall03()` menjadi `true`.

![Fungsi chall03()](/assets/img/posts/fridalab-walkthrough/12.png)
_Fungsi chall03()_

Berikut adalah *script* yang akan kita gunakan:

```javascript
Java.perform(function (){
 
  var a = Java.use("uk.rossmarks.fridalab.MainActivity");
  a.chall03.implementation = function() {
    return true;
  }

})
```

> Kita tidak perlu mengaitkan ke *class instance* `MainActivity` terlebih dahulu, karena fungsi `chall03()` akan dipanggil ketika kita menekan tombol `CHECK` dan kita hanya perlu meng-*hook* dan mengubah nilai pengembaliannya saja.
{: .prompt-tip }

![Berhasil Menyelesaikan Tantangan 3](/assets/img/posts/fridalab-walkthrough/13.png)
_Berhasil Menyelesaikan Tantangan 3_

## 0x6 - Send "frida" to chall04()

Untuk menyelesaikan tantangan 4, kita perlu memanggil fungsi `chall04()` dengan argumen string "**frida**".

![Fungsi chall04()](/assets/img/posts/fridalab-walkthrough/14.png)
_Fungsi chall04()_

Karena kita akan memanggil sebuah fungsi yang bukan statis, maka kita membutuhkan akses *class instance* `MainActivity`.

Berikut adalah *script* yang akan kita gunakan:

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        instance.chall04('frida'); //Calling the function with argument
      },
      onComplete: function() {}
    });
});
```

Jalankan *script* tersebut dan kita akan berhasil menyelesaikan tantangan 4.

![Berhasil Menyelesaikan Tantangan 4](/assets/img/posts/fridalab-walkthrough/15.png)
_Berhasil Menyelesaikan Tantangan 4_

## 0x7 - Always send "frida" to chall05()

Setiap kali kita menekan tombol `CHECK`, fungsi `chall05()` dengan argumen string "**notfrida!**" akan dipanggil.

![Pemanggilan Funsi chall05()](/assets/img/posts/fridalab-walkthrough/16.png)
_Pemanggilan Funsi chall05()_

Untuk menyelesaikan tantangan ini, kita harus meng-*hook* fungsi `chall05()` dan mengubah nilai argumennya menjadi "**frida**".

![Fungsi chall05()](/assets/img/posts/fridalab-walkthrough/17.png)
_Fungsi chall05()_

Berikut adalah *script* yang akan kita gunakan:

```javascript
Java.perform(function (){
 
  var a = Java.use("uk.rossmarks.fridalab.MainActivity");
  a.chall05.overload('java.lang.String').implementation = function(a){ //Calling the function with argument
    this.chall05('frida');
  }

})
```

Jalankan *script* tersebut dan kita akan berhasil menyelesaikan tantangan 5.

![Berhasil Menyelesaikan Tantangan 5](/assets/img/posts/fridalab-walkthrough/18.png)
_Berhasil Menyelesaikan Tantangan 5_

## 0x8 - Run chall06() after 10 seconds with correct value

Saat aplikasi berjalan, beberapa tindakan dilakukan, di antaranya:
- Fungsi `startTime()` dari kelas `challenge_06` dipanggil untuk menyimpan waktu saat ini dalam format *epoch time* ke dalam variabel `timeStart`.
- Dijadwalkan sebuah `TimerTask` untuk menjalankan kode yang secara teratur menghasilkan nilai acak dan menambahkannya ke variabel `chall06` setiap detiknya melalui fungsi `addChall06()`.
- Dalam fungsi `addChall06()`, jika nilai variabel `chall06` melebihi 9000, maka nilainya akan direset kembali.

![Pemanggilan Fungsi Ketika Aplikasi Berjalan](/assets/img/posts/fridalab-walkthrough/19.png)
_Pemanggilan Fungsi Ketika Aplikasi Berjalan_

![Class challenge_06](/assets/img/posts/fridalab-walkthrough/20.png)
_Class challenge_06_

![Fungsi chall06()](/assets/img/posts/fridalab-walkthrough/21.png)
_Fungsi chall06()_

Untuk menyelesaikan tantangan ini, kita perlu memanggil `chall06()` dengan menyertakan nilai argumen dari variabel `chall06` setidaknya setelah 10 detik aplikasi berjalan. Validasi ini diimplementasikan dalam fungsi `confirmChall06()`.

Setelah memahami kondisinya, jalankan aplikasi dan tunggu selama 10 detik. Setelah itu, jalankan *script* berikut:

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        var a = Java.use("uk.rossmarks.fridalab.challenge_06");
        instance.chall06(a.chall06.value); //Calling the function
      },
      onComplete: function() {}
    });
});
```

![Berhasil Menyelesaikan Tantangan 6](/assets/img/posts/fridalab-walkthrough/22.png)
_Berhasil Menyelesaikan Tantangan 6_

Yey.. Kita berhasil menyelesaikan tantangan 6.

Selain menunggu selama 10 detik sebelum menjalankan *script*, kita juga bisa menambahkan *delay* sebelum suatu kode dieksekusi. Dengan demikian kita bisa menyelesaikan tantangan ini dengan menjalankan *script* di awal pada saat aplikasi mulai berjalan.

Berikut adalah *script* yang akan kita gunakan, dengan penundaan (*delay*) selama 15 detik sebelum *script* dijalankan.

```javascript
setTimeout(function() {
    Java.performNow(function(){
        Java.choose('uk.rossmarks.fridalab.MainActivity', {
          onMatch: function(instance) { //"instance" is the instance for the MainActivity
            var a = Java.use("uk.rossmarks.fridalab.challenge_06");
            instance.chall06(a.chall06.value); //Calling the function
            console.log("Check now!")
          },
          onComplete: function() {}
        });
    });
}, 15000); //execute after 15 seconds
```

Simpan *script* tersebut dalam sebuah file, lalu jalankan aplikasi dan *script* menggunakan Frida.

```bash
➜ frida -U -f uk.rossmarks.fridalab -l script.js
```

![Berhasil Menyelesaikan Tantangan 6 Menggunakan Delay](/assets/img/posts/fridalab-walkthrough/23.png)
_Berhasil Menyelesaikan Tantangan 6 Menggunakan Delay_

Tantangan 6 berhasil diselesaikan dengan menyisipkan kode *delay*.

## 0x9 - Bruteforce check07Pin() then confirm with chall07()

Ketika aplikasi pertama kali berjalan, fungsi `setChall07()` dari *class* `challenge_07` akan dipanggil.

![Pemanggilan Fungsi Ketika Aplikasi Berjalan](/assets/img/posts/fridalab-walkthrough/24.png)
_Pemanggilan Fungsi Ketika Aplikasi Berjalan_

Fungsi `setChall07()` akan menyimpan nomor acak dengan rentang 1000 hingga 9000 pada variabel statis `chall07`.

![Class challenge_07](/assets/img/posts/fridalab-walkthrough/25.png)
_Class challenge_07_


Untuk menyelesaikan tantangan ini, kita perlu memanggil fungsi `chall07()` pada *class* `MainActivity` dengan argumen yang nilainya sama dengan nomor acak yang telah tersimpan.

![Fungsi chall07()](/assets/img/posts/fridalab-walkthrough/26.png)
_Fungsi chall07()_

Ada beberapa cara untuk menyelesaikan tantangan ini. Salah satunya adalah dengan menggunakan pendekatan seperti pada [tantangan 6](#0x8---run-chall06-after-10-seconds-with-correct-value) yang langsung mengakses variabel statis sebagai nilai argumen dalam pemanggilan fungsi `chall07()`. Berikut contoh *script*-nya:

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        var a = Java.use("uk.rossmarks.fridalab.challenge_07");
        console.log("chall07: " + a.chall07.value);
        instance.chall07(a.chall07.value); //Calling the function
      },
      onComplete: function() {}
    });
});
```

![Berhasil Menyelesaikan Tantangan 7](/assets/img/posts/fridalab-walkthrough/27.png)
_Berhasil Menyelesaikan Tantangan 7_

Dari gambar di atas, terlihat bahwa tantangan 7 berhasil diselesaikan.

Selain itu, kita juga dapat menggunakan metode *bruteforce* sebagaimana yang disiratkan dalam judul tantangan. Pada pendekatan ini, kita mencoba PIN dari 1000 hingga 9000. Berikut contoh *script*-nya:

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        var a = Java.use("uk.rossmarks.fridalab.challenge_07");

        // brute force
        for (i=1000; i<=9000; i++) {
            if (a.check07Pin(i.toString())) {
                console.log("Pin: " + a.chall07.value);
                instance.chall07(a.chall07.value);
            }
        }

      },
      onComplete: function() {}
    });
});
```

![Berhasil Menyelesaikan Tantangan 7 Dengan Cara Bruteforce PIN](/assets/img/posts/fridalab-walkthrough/28.png)
_Berhasil Menyelesaikan Tantangan 7 Dengan Cara Bruteforce PIN_

Dengan pendekatan *bruteforce* PIN, tantangan 7 berhasil diselesaikan.

## 0xA - Change 'check' button's text value to 'Confirm'

Setiap tombol `CHECK` ditekan, fungsi `chall08()` akan dipanggil dan mengembalikan nilai `true` atau `false`.

![Pemanggilan Fungsi Ketika Aplikasi Berjalan](/assets/img/posts/fridalab-walkthrough/29.png)
_Pemanggilan Fungsi Ketika Aplikasi Berjalan_

Untuk menyelesaikan tantangan ini, nilai pengembalian dari fungsi `chall08` harus menjadi `true`. Fungsi ini akan mengembalikan nilai `true` jika teks tombol diubah dari `CHECK` menjadi `Confirm`.

![Fungsi chall08()](/assets/img/posts/fridalab-walkthrough/30.png)
_Fungsi chall08()_

Kita bisa menggunakan Frida untuk mengubah nilai pengembalian seperti pada [tantangan 3](#0x5---make-chall03-return-true). Berikut contoh *script*-nya:

```javascript
Java.perform(function (){
 
    var a = Java.use("uk.rossmarks.fridalab.MainActivity");
    a.chall08.implementation = function(){

		console.log("This method is hooked");
		return true // set return "true"
        
    }

})
```

![Berhasil Menyelesaikan Tantangan 8](/assets/img/posts/fridalab-walkthrough/31.png)
_Berhasil Menyelesaikan Tantangan 8_

Dari gambar di atas, terlihat bahwa tantangan 8 berhasil diselesaikan.

Selain itu, kita juga bisa menggunakan cara mengubah teks tombol `CHECK` menjadi `Confirm` sesuai dengan yang disiratkan dalam judul tantangan.

Untuk mengubah tombol `CHECK`, kita perlu menggunakan fungsi `findViewById` yang ada di *class instance* `MainActivity`. `findViewById()` berfungsi untuk mencari dan mengambil referensi ke *view* (komponen UI) di layout XML.

Mari kita siapkan *template script* untuk mengakses *class instance* `MainActivity`.

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        // TO DO
      },
      onComplete: function() {}
    });
});
```

Selanjutnya, kita siapkan objek `button` untuk berinteraksi dengan *class* `android.widget.Button`.

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) {
        var button = Java.use('android.widget.Button'); //set up a button object to interact with Java's android.widget.Button class.
      },
      onComplete: function() {}
    });
});
```

`R` adalah *class* pada Android yang memiliki ID dari semua komponen UI. Komponen yang digunakan pada tombol `CHECK` adalah `R.id.check`. Jika kita mengklik dua kali pada kode tersebut, kita akan melihat bahwa komponen ini memiliki ID `0x7f07002f`.

![Nilai R.id.check](/assets/img/posts/fridalab-walkthrough/32.png)
_Nilai R.id.check_

Setelah mengetahui ID dari komponen yang akan diubah. kita melakukan *casting* untuk memastikan bahwa objek `checkid` adalah *instance* dari *class* `android.widget.Button`.

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) {
        var button = Java.use('android.widget.Button');
        var checkid = instance.findViewById(0x7f07002f); //find a view with ID 0x7f07002f (the Button that we want to modify).
        var check = Java.cast(checkid, button); //ensure that the view found is Button by casting to the android.widget.Button class.
      },
      onComplete: function() {}
    });
});
```

Selanjutnya, kita siapkan objek `string` untuk menangani string dalam Java dan memanggil method `setText` pada tombol untuk mengubah teks tombol menjadi "`Confirm`".

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) {
        var button = Java.use('android.widget.Button');
        var checkid = instance.findViewById(0x7f07002f);
        var check = Java.cast(checkid, button);
        var string = Java.use('java.lang.String'); //set up a string object to handle strings in Java.
        check.setText(string.$new("Confirm")); //calls the setText method on the Button to change the Button text to "Confirm"
      },
      onComplete: function() {}
    });
});
```

Berikut adalah *script* lengkapnya:

```javascript
Java.performNow(function(){
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
      onMatch: function(instance) { //"instance" is the instance for the MainActivity
        var button = Java.use('android.widget.Button'); //set up a button object to interact with Java's android.widget.Button class.
        var checkid = instance.findViewById(0x7f07002f); //find a view with ID 0x7f07002f (the Button that we want to modify).
        var check = Java.cast(checkid, button); //ensure that the view found is Button by casting to the android.widget.Button class.
        var string = Java.use('java.lang.String'); //set up a string object to handle strings in Java.
        check.setText(string.$new("Confirm")); //calls the setText method on the Button to change the Button text to "Confirm"
      },
      onComplete: function() {}
    });
});
```

![Berhasil Mengubah Teks Tombol](/assets/img/posts/fridalab-walkthrough/33.png)
_Berhasil Mengubah Teks Tombol_

Kita berhasil mengubah teks tombol dan menyelesaikan tantangannya.