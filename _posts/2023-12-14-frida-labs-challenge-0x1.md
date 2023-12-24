---
title: '[Frida Labs] 01 - Frida Setup & Hooking a Method'
date: 2023-12-14 09:53:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---


Dalam bagian ini, kita akan fokus pada pemahaman, persiapan, dan memulai proses *hooking method* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x1.apk**, yang bisa diunduh dari [sini](https://github.com/DERE-ad2001/Frida-Labs/raw/main/Frida%200x1/Challenge%200x1.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.

## 0x2 - What is Frida?

Frida adalah *magic tool* untuk komputer atau perangkat seluler. Frida bisa membantu kita melihat apa yang terjadi di dalam suatu program atau aplikasi, meskipun kita tidak memiliki kode aslinya. 

Frida juga bisa mengintervensi fungsi-fungsi dalam program. Jadi, seperti memberi kita kemampuan untuk mengubah atau mengamati cara kerja aplikasi dari dalam, seperti:

- **Intercepting Function Calls**: Frida memungkinkan kita untuk mengidentifikasi fungsi atau metode tertentu dalam aplikasi dan menginterupsinya. Ketika fungsi-fungsi ini dipanggil, Frida bisa mengubah data yang diterima atau melihat apa yang sedang dilakukan.
- **Observing and Modifying**: Kita bisa mengamati apa yang terjadi di dalam program secara *real-time*. Misalnya, kita bisa melihat nilai-nilai variabel, memahami alur program, bahkan mengubah data atau kode saat sedang dieksekusi.
- **Debugging and Reverse Engineering**: Kemampuan ini sangat berguna untuk *debugging*, *reverse engineering*, dan *security analysis*. Digunakan oleh *developer* untuk mendiagnosis dan memperbaiki masalah di aplikasi mereka, serta oleh *security experts* untuk mengungkap kerentanan dan ancaman potensial.
- **Dynamic Analysis**: Berbeda dengan alat *debugging* tradisional, Frida tidak memerlukan akses ke kode sumber asli. Frida bisa bekerja dengan kode yang telah dikompilasi, membuatnya sangat berguna untuk memeriksa aplikasi *closed-source*.

## 0x3 - Getting Started with Frida

Untuk menggunakan Frida, kita perlu menginstal `frida-tools` pada komputer kita dan menjalankan `frida-server` pada perangkat Android.

Instalasi Frida dapat dilakukan menggunakan `pip`:

```bash
➜ pip install frida-tools

➜ frida --version                            
16.1.8
```
{: .nolineno }

Saat ini, saya menggunakan Frida versi 16.1.8.

Langkah berikutnya adalah menjalankan Frida server di perangkat Android. Pertama, unduh Frida server versi 16.1.8 (sesuaikan dengan versi Frida) dari [situs resminya](https://github.com/frida/frida/releases). Pastikan untuk menggunakan Frida server yang cocok dengan arsitektur Android yang digunakan.

Jika kamu tidak yakin tentang arsitektur Android yang digunakan, gunakan perintah `adb` berikut:

```bash
➜ adb shell getprop ro.product.cpu.abi
```
{: .nolineno }

![Melihat Arsitektur Android](/assets/img/posts/frida-labs-challenge-0x1/2.png)
_Melihat Arsitektur Android_

Karena saya menggunakan perangkat `arm64-v8a`, saya memilih versi **Frida server arm64**.

![List Arsiektur Yang Didukung Frida Server](/assets/img/posts/frida-labs-challenge-0x1/1.png)
_List Arsiektur Yang Didukung Frida Server_

Setelah diunduh, ekstrak file tersebut dan pindahkan ke direktori yang *writeable*, seperti `/data/local/tmp`:

```bash
➜ adb push frida-server-16.1.8-android-arm64 /data/local/tmp
```
{: .nolineno }

Kemudian, akses direktori tersebut melalui *shell*:

```bash
➜ adb shell
RMX1851:/ $ su
RMX1851:/ # cd /data/local/tmp 
RMX1851:/data/local/tmp # ls
frida-server-16.1.8-android-arm64
```
{: .nolineno }

Berikan hak akses eksekusi pada binary `frida-server`:

```bash
RMX1851:/data/local/tmp # chmod +x frida-server-16.1.8-android-arm64
```
{: .nolineno }

Sekarang jalankan Frida server:

```bash
RMX1851:/data/local/tmp # ./frida-server-16.1.8-android-arm64
```
{: .nolineno }

![Menjalankan Frida Server](/assets/img/posts/frida-labs-challenge-0x1/3.png)
_Menjalankan Frida Server_

Terlihat pada gambar di atas, Frida server berjalan dengan baik. Jika kamu menemukan kesalahan, saya sarankan untuk mencoba mencari solusinya di Google.
## 0x4 - Basic usage Frida

Jika kamu ingin melihat daftar aplikasi yang terpasang di perangkat Android, gunakan perintah berikut:

```bash
➜ frida-ps -Uai
➜ frida-ps -Uai | grep '<name_of_application>'
```
{: .nolineno }

- `frida-ps`:  Menampilkan informasi tentang proses yang sedang berjalan pada perangkat Android.
- `-U`: Menghubungkan Frida ke perangkat melalui USB (perangkat fisik atau emulator).
- `-a`: Menampilkan semua aplikasi yang sedang berjalan.
- `-i`: Menampilkan semua aplikasi yang terinstal.

![Daftar Aplikasi Yang Terinstal](/assets/img/posts/frida-labs-challenge-0x1/4.png)
_Daftar Aplikasi Yang Terinstal_

Untuk mengaitkan Frida dengan aplikasi, kita memerlukan *identifier* aplikasi tersebut. Setelah mendapatkan *identifier*, kita dapat mengaitkan Frida seperti ini:

```bash
➜ frida -U -f <package_name>
```
{: .nolineno }

Saya akan mencoba mengaitkan Frida dengan aplikasi **Challenge 0x1.apk**. *Identifier* dari aplikasi ini adalah `com.ad2001.frida0x1`.

```bash
➜ frida -U -f com.ad2001.frida0x1
```
{: .nolineno }

![Menjalankan Aplikasi Dengan Frida](/assets/img/posts/frida-labs-challenge-0x1/5.png)
_Menjalankan Aplikasi Dengan Frida_

Sekarang, aplikasi telah dijalankan dan berhasil terhubung dengan Frida.

## 0x5 - Introduction to Hooking

**Hooking** merujuk pada proses intersepsi dan modifikasi perilaku fungsi atau metode dalam aplikasi. Sebagai contoh, kita dapat mengaitkan (*hook*) sebuah metode dalam aplikasi dan mengubah fungsionalitasnya sesuai keinginan.

Sekarang, mari kita coba meng-*hook* sebuah metode dalam sebuah aplikasi. Kita akan melakukan ini menggunakan API JavaScript, tetapi perlu dicatat bahwa Frida juga mendukung Python.

### A. Challenge 0x1 App

Aplikasi studi kasus kali ini memiliki *Identifier* `com.ad2001.frida0x1`.

Sebelum meng-*hook* aplikasi menggunakan Frida, mari kita luangkan waktu untuk memahami aplikasi. Setelah membuka aplikasi, kita dapat melihat antarmuka di bawah ini:

![Tampilan Aplikasi](/assets/img/posts/frida-labs-challenge-0x1/6.png)
_Tampilan Aplikasi_

Aplikasi ini meminta kita untuk memasukkan sebuah angka. Mari kita masukkan angka dan lihat.

![Mencoba Memasukkan Angka](/assets/img/posts/frida-labs-challenge-0x1/7.png)
_Mencoba Memasukkan Angka_

Tertulis `Try again`. Jadi, mari kita coba dekompilasi aplikasi menggunakan `JADX`.

![Reverse Engineering Aplikasi Menggunakan JADX](/assets/img/posts/frida-labs-challenge-0x1/8.png)
_Reverse Engineering Aplikasi Menggunakan JADX_

Hanya dengan sekilas melihat kode Java, kita dapat memahami bahwa aplikasi mengambil input-an pengguna, mengubahnya menjadi integer, dan mengirimkan integer tersebut ke metode `check()`.

```java
public void onClick(View view) {
	String obj = editText.getText().toString();
	if (TextUtils.isDigitsOnly(obj)) {
		MainActivity.this.check(i, Integer.parseInt(obj));
	} else {
		Toast.makeText(MainActivity.this.getApplicationContext(), "Enter a valid number !!", 1).show();
	}
}
```

Bersama dengan integer yang di-input-kan, nilai integer lainnya juga dikirimkan ke metode `check()`.

![Fungsi check()](/assets/img/posts/frida-labs-challenge-0x1/9.png)
_Fungsi check()_

Ketika aplikasi dimulai, fungsi `get_random()` menghasilkan sebuah nilai acak yang berada dalam kisaran 0 hingga 100, dan nilai ini disimpan dalam variabel `i`. Fungsi ini hanya dipanggil satu kali, sehingga nomor acak yang dihasilkan tidak akan berubah selama aplikasi berjalan. Namun, setiap kali aplikasi dijalankan ulang, sebuah nomor acak yang berbeda akan dihasilkan.

Sekarang mari kita lihat apa yang terjadi dalam fungsi `check()`.

```java
void check(int i, int i2)
```

Di sini `i` merujuk pada nomor acak yang dihasilkan oleh fungsi `get_random()` dan `i2` merujuk pada nomor integer yang di-input-kan pengguna.

```java
if ((i * 2) + 4 == i2)
```

Pernyataan `if` ini memeriksa apakah nomor yang di-input-kan sama dengan (**nilai acak * 2 + 4**). Jika cocok, maka aplikasi akan mendekode *flag* dan menampilkannya di `TextView`. 

Untuk mendapatkan *flag*, kita perlu menemukan nomor acak dan melakukan operasi aritmatika yang ditentukan, kemudian memasukkan hasilnya ke dalam aplikasi.

Kita bisa dengan mudah mendapatkan *flag* menggunakan metode alternatif, tetapi tujuan utama kita di sini adalah untuk mengenal Frida. Untuk mencapainya, kita memerlukan cara untuk mendapatkan nomor acak menggunakan Frida, dan ada beberapa cara untuk melakukan ini:

- Meng-*hook* fungsi `get_random()`.
	- Karena kita tahu bahwa nomor acak dihasilkan dalam metode `get_random()`, kita dapat meng-*hook* metode ini untuk mendapatkan *return value*, atau kita dapat menimpa *return value*-nya dengan nilai yang kita tentukan untuk dikirimkan ke fungsi `check()`.
- Meng-*hook* fungsi `check()`.
	- Argumen yang dikirimkan ke metode `check()` mengandung nomor acak. Dengan demikian, kita dapat mencoba mengaitkan metode ini untuk mengambil argumen dan menemukan nomor acak.

Sekarang kita tahu bagaimana cara menyelesaikannya, mari kita coba menulis beberapa *script* Frida.

### B. Hooking a method

Pertama, izinkan saya memberikan sebuah template *script*, kemudian saya akan menjelaskannya.

```javascript
Java.perform(function (){
 
    var <class_reference> = Java.use("<package_name>.<class>");
    <class_reference>.<method_to_hook>.implementation = function(<args>){

    /*
        OUR OWN IMPLEMENTATION OF THE METHOD
    */
        
    }

})
```
{: file="script.js" }

- `Java.perform` adalah fungsi dalam Frida yang digunakan untuk menciptakan konteks khusus bagi *script* kita untuk berinteraksi dengan kode Java dalam aplikasi Android. Ini seperti membuka pintu untuk mengakses dan memanipulasi kode Java yang berjalan di dalam aplikasi. Setelah berada di dalam konteks ini, kita dapat melakukan tindakan seperti meng-*hook* metode atau mengakses *class* Java untuk mengendalikan atau mengamati perilaku aplikasi.
- `var <class_reference> = Java.use("<package_name>.<class>");`
  Di sini, kita mendeklarasikan variabel `<class_reference>` untuk mewakili *class* Java dalam aplikasi Android target. Kita menentukan *class* yang akan digunakan dengan fungsi `Java.use`, yang mengambil nama *class* sebagai argumen. `<package_name>` mewakili nama *package* dari aplikasi Android, dan `<class>` mewakili *class* yang ingin kita interaksikan.
- `<class_reference>.<method_to_hook>.implementation = function(<args>) {}`
  Di dalam *class* terpilih, kita menentukan metode yang ingin kita *hook* dengan mengaksesnya menggunakan notasi `<class_reference>.<method_to_hook>`. Di sinilah kita dapat mendefinisikan logika kita sendiri untuk dieksekusi ketika metode yang di-*hook* dipanggil. `<args>` mewakili argumen yang dikirim ke fungsi.

Sekarang pertanyaannya adalah, apa yang akan kita *hook*?

#### Hooking the get_random() method

Mari kita coba meng-*hook* metode `get_random()` kali ini. *Package* yang digunakan yaitu `com.ad2001.frida0x1`. 

Selanjutnya, kita perlu mengidentifikasi nama *class* di mana metode yang ingin kita *hook* berada.

![Mengidentifikasi Package Dan Class](/assets/img/posts/frida-labs-challenge-0x1/10.png)
_Mengidentifikasi Package Dan Class_

Seperti yang bisa kita lihat, kita harus mendapatkan referensi ke `MainActivity`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");

})
```
{: file="script.js" }

Selanjutnya, kita akan memodifikasi *script* untuk menyertakan implementasi kustom kita dari metode tersebut. Metode yang akan di-*hook* adalah `get_random`.

```java
int get_random() {
	return new Random().nextInt(100);
}
```

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.get_random.implementation = function(){

        console.log("This method is hooked");
        
    }

})
```
{: file="script.js" }

Ketika kita menjalankan *script* ini, ia akan meng-*hook* fungsi `get_random()`. Ini berarti bahwa setiap kali fungsi `get_random()` dipanggil, kode *script* kita akan dieksekusi menggantikan yang asli. Dalam kasus ini, ketika metode dipanggil, ia akan mencetak "**This method is hooked**". *Script* ini masih belum selesai, dan perhatikan bahwa saya tidak memasukkan argumen apa pun dalam `function()` karena `get_random()` tidak memerlukan argumen apa pun.

Sekarang, mari kita jalankan *script* untuk mengamati perilakunya.

Pertama, mari kita *hook* aplikasi dengan Frida.

```bash
➜ frida -U -f com.ad2001.frida0x1
```
{: .nolineno }

Oke, sekarang Frida telah terhubung. Untuk menjalankan *script*, cukup *copy* dan *paste* ke *console*, seperti berikut, lalu tekan **Enter**.

![Menjalankan Kode Javascript Di Console Frida](/assets/img/posts/frida-labs-challenge-0x1/11.png)

Jika *script* tidak memiliki kesalahan, seharusnya ditampilkan seperti di atas. Jika terjadi kesalahan, Frida akan memberikan peringatan, dan kita harus memeriksa *script*-nya kembali.

Mari kita coba memasukkan angka.

![Mencoba Memasukkan Angka](/assets/img/posts/frida-labs-challenge-0x1/12.png)
_Mencoba Memasukkan Angka_

Aplikasi menampilkan pesan `Try again`. Jelas karena kita tidak tahu angka yang benar. Namun, ketika kita memeriksa *console* Frida, kita tidak melihat informasi atau *output* apa pun.

Alasannya adalah fungsi `get_random()` dijalankan ketika aplikasi diluncurkan. Sedangkan kita menyisipkan *script* setelah `get_random()` telah dieksekusi. Jika kita melihat dekompilasi, kita bisa memahaminya.

Jadi, apa yang akan kita lakukan?

Kita perlu menyisipkan *script* pada saat yang sama saat aplikasi dimuat, memungkinkan kita untuk meng-*hook* metode ini sebelum dieksekusi. Untuk melakukan ini, kita bisa menggunakan opsi `-l`. Pertama, mari kita simpan *script* kita ke dalam sebuah file.

Saya menyimpan *script* pada file `script.js`. Sekarang mari kita muat *script* ini menggunakan opsi `-l`.

```bash
➜ frida -U -f com.ad2001.frida0x1 -l script.js
```
{: .nolineno }

![Menjalankan Frida Dengan File Script](/assets/img/posts/frida-labs-challenge-0x1/13.png)
_Menjalankan Frida Dengan File Script_

Kita telah berhasil meng-*hook* metode `get_random()` tetapi kita mendapatkan kesalahan. Tertulis bahwa `get_random()` mengharapkan *return value*. Jika kita melihat implementasi `get_random()`, ia mengembalikan sebuah angka.

```java
int get_random() {
	return new Random().nextInt(100);
}
```

Dalam *script* kita, kita menggantikan implementasi asli dari metode `get_random()` dengan yang kustom, tetapi kita tidak memberikan *return value*. *Return value* ini ditugaskan ke variabel `i` dan digunakan dalam fungsi `check()`. Jadi, mari kita coba memberikan *return value*. Kita bisa menggunakan nilai apa saja.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.get_random.implementation = function(){

        console.log("This method is hooked");
        console.log("Returning 5")

        return 5;
        
    }

})
```
{: file="script.js" }

Saya menggunakan nilai `5`. Sekarang jika kita menyisipkan kode ini, fungsi `get_random()` akan mengembalikan `5`.

Mari kita coba menjalankan *script*.

```bash
➜ frida -U -f com.ad2001.frida0x1 -l script.js
```
{: .nolineno }


Kita dapat melihat bahwa tidak ada kesalahan yang muncul di sini; metode tersebut telah dipanggil dan mengembalikan nilai `5`.

Sekarang, `5` akan dikirimkan ke fungsi `check()`. Mari kita hitung nilai agar kita bisa memenuhi pemeriksaan `if` dan mendapatkan *flag*.

```java
if ((i * 2) + 4 == i2)
```

Jadi, **5 * 2 + 4** sama dengan **14**. Jika kita memasukkan `14` pada input, kita bisa mendapatkan *flag*. Mari kita coba.

![Berhasil Mengubah Return Value get_random()](/assets/img/posts/frida-labs-challenge-0x1/14.png)
_Berhasil Mengubah Return Value get_random()_

Lihat! Kita berhasil mendapatkan *flag*-nya.

Sekarang, mari kita coba untuk mengambil nilai acak asli yang dihasilkan. Untuk mencapai ini, kita perlu mendapatkan *return value* dari fungsi `get_random()` asli. Mari kita lihat bagaimana melakukannya.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.get_random.implementation = function(){

        console.log("This method is hooked");
        var ret_val = this.get_random();
        console.log("The return value is "+ ret_val);   
    }

})
```
{: file="script.js" }

Apa yang kita lakukan di sini adalah meng-*hook* metode `get_random()`. Dalam *hook* ini, kita memanggil `get_random()` asli dengan menggunakan `this.get_random()`. Kata kunci `this` merujuk pada objek saat ini. Karena metode ini menghasilkan *return value* asli, kita menyimpannya dalam variabel `ret_val`. Namun, jika kita menjalankan *script* ini, aplikasi akan mengalami *crash* karena `get_random` harus memberikan *return value*. Oleh karena itu, kita dapat mengembalikan nilai asli dan untuk mengatasi pemeriksaan, kita dapat memanfaatkan nilai acak asli yang tersimpan dalam `ret_val`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.get_random.implementation = function(){

        console.log("This method is hooked");
        var ret_val = this.get_random();
        console.log("The return value is "+ ret_val);
        console.log("The value to bypass the check is " + (ret_val * 2 + 4 ))

        return ret_val; //returning the original random value from the get_random method
    }

})
```
{: file="script.js" }

Mari kita simpan dan jalankan *script* ini.

```bash
➜ frida -U -f com.ad2001.frida0x1 -l script.js
```
{: .nolineno }

![Menampilkan Angka Random Asli](/assets/img/posts/frida-labs-challenge-0x1/15.png)
_Menampilkan Angka Random Asli_

Nilai acak yang dihasilkan adalah `9`. *script* kita juga menghitung nilai untuk menghindari pemeriksaan. Jadi mari kita coba memasukkan nilai `22`.

Lihat! Kita berhasil mendapatkan *flag*-nya.

#### Hooking the check() method

Mari kita coba metode kedua yang saya sebutkan [di awal](#a-challenge-0x1-app). Kita akan meng-*hook* ke metode `check()` dan menangkap argumennya, karena argumen yang dikirimkan ke metode `check()` mengandung nomor acak.

```java
final int i = get_random();
...
...

void check(int i, int i2) {
        if ((i * 2) + 4 == i2) {
        ...
        ...
}
```

Jika kita memeriksa argumen untuk fungsi `check`, argumen pertama, `i`, adalah nomor acak, sedangkan yang kedua, `i2`, sesuai dengan nomor yang dimasukkan pengguna. Mari kita tangkap dan *dump* kedua argumen ini menggunakan Frida.

Saat meng-*hook* metode yang memiliki argumen, penting untuk menentukan tipe argumen yang diharapkan menggunakan kata kunci `overload(arg_type)`. Selain itu, pastikan kita menyertakan argumen yang ditentukan ini dalam implementasi kita saat meng-*hook* metode tersebut. Di sini, fungsi `check()` kita memerlukan dua argumen integer, sehingga kita dapat menentukannya seperti ini:

```javascript
a.check.overload('int' ,'int').implementation = function(a,b){
	...
}
```
{: file="script.js" }

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.check.overload('int' ,'int').implementation = function(a,b){ //The function takes two arguments ;check(random,input)
        console.log("The random number is "+a);
        console.log("The user input is "+b);
    }

})
```
{: file="script.js" }

Setelah mendapatkan argumen ini, tujuan utamanya adalah untuk memastikan bahwa fungsi `check` tetap berfungsi normal karena mengandung kode untuk menghasilkan *flag*. Tujuan utama saat ini adalah untuk mengekstrak nilai acak tanpa mengganggu fungsionalitas keseluruhan fungsi. Jadi, kita bisa sekadar memanggil fungsi `check()` asli, seperti yang kita lakukan di atas dengan fungsi `get_random`. Jangan lupa untuk meneruskan argumen ke pemanggilan `check()` asli.

```javascript
Java.perform(function() {
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.check.overload('int', 'int').implementation = function(a, b) {
        // The function takes two arguments; check(random, input)
        console.log("The random number is " + a);
        console.log("The user input is " + b);
        this.check(a, b); // Call the check() function with the correct arguments
    }
});
```
{: file="script.js" }

Mari kita coba menjalankan *script* ini. Kita tidak perlu memuat *script* ini di awal (menggunakan opsi `-l`), karena fungsi `check()` hanya dipanggil ketika kita mengklik tombol. 

```bash
frida -U -f com.ad2001.frida0x1
```
{: .nolineno }

Mari kita masukkan input dan klik tombol kirim.

![Mendapatkan Angka Acak Melalui Fungsi check()](/assets/img/posts/frida-labs-challenge-0x1/16.png)
_Mendapatkan Angka Acak Melalui Fungsi check()_

Kita melihat bahwa nomor acak yang dihasilkan adalah `6`. Jadi memasukkan **(6 * 2 + 4)** `16` akan memberi kita *flag*.

Sebelum mengakhiri contoh ini, saya ingin menunjukkan satu cara lain untuk mendapatkan *flag*.

Kita tahu bahwa untuk mendapatkan *flag*, input kita harus sesuai dengan hasil dari **(nomor acak * 2 + 4)**. Jadi, mengapa tidak mencoba memanggil fungsi `check()` dengan dua angka yang memenuhi kondisi ini? Dengan cara ini, kita tidak perlu memikirkan nomor acak karena kita yang memberikan input kita sendiri ke fungsi `check()`.

Mari kita coba. Kita akan memasukkan angka `4` sebagai input kita dan **(4 * 2 + 4)** sebagai argumen kedua, yang sama dengan `12`.

```javascript
Java.perform(function (){
 
    var a = Java.use("com.ad2001.frida0x1.MainActivity");
    a.check.overload('int' ,'int').implementation = function(a,b){ //The function takes two arguments ;check(random,input)
        this.check(4, 12); 
    }

})
```
{: file="script.js" }

![Menentukan Nilai Argumen Fungsi Dengan Frida](/assets/img/posts/frida-labs-challenge-0x1/17.png)
_Menentukan Nilai Argumen Fungsi Dengan Frida_

Seperti yang kita harapkan, kita mendapatkan *flag*.

Penjelasan di atas adalah prinsip dasar meng-*hook* metode di Frida dan men-*dump* argumen serta *return value*-nya. Frida adalah alat yang sangat *powerful*, dan kita akan mengeksplorasi beberapa fitur utamanya lebih lanjut dalam pembahasan [Frida Labs](/tags/frida-labs/).
