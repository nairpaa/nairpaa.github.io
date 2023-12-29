---
title: '[Frida Labs] 11 - Patching Instructions using X86Writer and ARM64Writer'
date: 2023-12-29 18:07:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam bagian ini, kita akan mempelajari cara *patching* intruksi menggunakan `X86Writer` dan `ARM64Writer`.

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0xB.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200xB/Challenge%200xB.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.
- *Basics of x64/ARM64 assembly and reversing*.

## 0x2 - Challenge 0xB

Tantangan ini bertujuan untuk mengenalkan *temporary patching* menggunakan Frida. Mari kita mulai dengan menginstal dan membuka aplikasi.

![Aplikasi Challenge 0xB](/assets/img/posts/frida-labs-challenge-0xb/1.png)
_Aplikasi Challenge 0xB_

Hanya ada tombol, dan ketika diklik tidak menghasilkan respons apa pun. Mari kita dekompilasi aplikasi menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0xb/2.png)
_Class MainActivity_

Di bagian atas kode, kita temukan deklarasi fungsi *native* `getFlag()`.

```java
static {
	System.loadLibrary("frida0xb");
}
```

Dan aplikasi memanggil *library* `frida0xb` di bagian bawah kode menggunakan `System.loadLibrary()`. 

Dalam `onCreate`, aplikasi memanggil fungsi *native* `getFlag()`, yang tidak mengambil argumen serta tidak mengembalikan nilai apa pun.

Mari kita analisis *library* tersebut menggunakan Ghidra. Kita akan mulai dengan versi x86 sebelum ARM64.

![Daftar Library](/assets/img/posts/frida-labs-challenge-0xb/3.png)
_Daftar Library_

Kita muat `libfrida0xb.so` ke Ghidra dan memeriksa fungsi `getFlag()`. 

![Fungsi getFlag()](/assets/img/posts/frida-labs-challenge-0xb/4.png)
_Fungsi getFlag()_

Jika kita lihat hasil dekompilasi, kodenya aneh dan tidak masuk akal.

Tapi jika kita memeriksa bagian *disassembly*, kita bisa melihat alasannya.

![Disassembly getFlag() x86](/assets/img/posts/frida-labs-challenge-0xb/5.png)
_Disassembly getFlag() x86_

Kode ini memuat `0xdeadbeef` ke variabel `local_14` dan membandingkannya dengan `0x539`, yang jelas berbeda. Dari sini kita bisa menduga blok kode berikutnya hanya dijalankan jika perbandingan ini benar. 

Secara default, Ghidra melakukan pengoptimalan dekompilasi, yang membuat ia melewatkan bagian kode ini karena perbandingannya tidak akan benar. Oleh karena itu, kita bisa menonaktifkan optimalisasi ini pada menu `Graph`.

![Ghidra Graph](/assets/img/posts/frida-labs-challenge-0xb/6.png)
_Ghidra Graph_

Pergi ke opsi `Edit` -> `Tool Options`.

![Ghidra Graph Tool Option](/assets/img/posts/frida-labs-challenge-0xb/7.png)
_Ghidra Graph Tool Option_

Hilangkan tanda centang pada `Eliminate unreachable code` dan klik `Apply`. 

![Pengaturan Ghidra](/assets/img/posts/frida-labs-challenge-0xb/8.png)
_Pengaturan Ghidra_

Sekarang mari kita periksa kembali hasil dekompilasinya.

![Hasil Dekompilasi Tanpa Optimalisasi](/assets/img/posts/frida-labs-challenge-0xb/9.png)
_Hasil Dekompilasi Tanpa Optimalisasi_

Meskipun dekompilasi tidak sepenuhnya akurat, kita bisa mengasumsikan `if(false)` ini seperti `if (local_14 == 1337)`. Selain itu, kita juga melihat kode mendekode string `j~ehmWbmxezisdmogi~Q` menggunakan operasi XOR dengan kunci `0x2c`, kemudian hasilnya dicetak dalam log. Kita dapat menduga bahwa string ini adalah *flag* yang perlu kita dapatkan.

Jadi, bagaimana kita bisa mendapatkan *flag* ini? Walapun fungsi `getFlag()` ini dipanggil saat kita menekan tombol di aplikasi, *flag* tidak akan pernah dicetak karena pemeriksaan `if` tidak pernah terpenuhi.

Dalam situasi ini, kita bisa mem-*patch* instruksi `JNZ` menjadi `NOP` menggunakan `x86Writer` untuk arsitektur x86 dan `Arm64Writer` untuk ARM64. Akibatnya, program akan mengabaikan kondisi lompatan dan melanjutkan untuk mengeksekusi perintah selanjutnya, yang memungkinkan pencetakan *flag* di log sesuai yang diharapkan.

## 0x3 - Patching using X86Writer

Berikut adalah *template script* untuk `x86Writer` yang akan kita gunakan:

```javascript
var writer = new x86Writer(<address_of_the_instruction>);

try {
  // Insert instructions

  // Flush the changes to memory
  writer.flush();

} finally {
  // Dispose of the Arm64Writer to free up resources
  writer.dispose();
}
```

**Instantiation of x86Writer:**

- `var writer = new x86Writer(<address_of_the_instruction>);`
- Baris ini membuat sebuah objek `x86Writer` baru dan menetapkan alamat instruksi yang ingin kita ubah. Ini mempersiapkan `writer` untuk bekerja pada alamat memori tertentu.

**Inserting instructions:**

- `try { /* Insert instructions here */ }`
- Dalam blok `try`, kita bisa menulis instruksi x86 yang ingin dimodifikasi atau ditambahkan. `x86Writer` menyediakan metode untuk menulis beragam instruksi x86 yang dapat kita lihat di dokumentasinya.

**Flushing the Changes :**

- `writer.flush();`
- Setelah instruksi dimasukkan, gunakan metode `flush` untuk menerapkan perubahan tersebut ke dalam memori. Ini memastikan instruksi baru tersimpan dan aktif di alamat yang dituju.

**Cleanup :**

- `finally { /* Dispose of the x86Writer to free up resources */ writer.dispose(); }`
- Gunakan blok `finally` untuk memastikan `x86Writer` dibersihkan setelah selesai digunakan. Memanggil metode `dispose` melepaskan *resource* yang digunakan oleh objek `x86Writer`.

Kita telah memiliki gambaran tentang *template script* tersebut. Selanjutnya, kita harus mencaari tahu intruksi apa yang ingin kita *patch*. Ini memerlukan pengetahuan dasar tentang *reverse engineering*. Untuk mempersingkat waktu, mari kita pertimbangkan tiga intruksi berikut:

```nasm
00020e1c c7  45  f0       MOV        dword ptr [EBP  + local_14 ],0xdeadbeef
		 ef  be  ad  de
00020e23 81  7d  f0       CMP        dword ptr [EBP  + local_14 ],0x539
		 39  05  00  00
00020e2a 0f  85  d8       JNZ        LAB_00020f08
		 00  00  00
```

Pertama, kode akan memasukkan nilai `0xdeadbeef` ke dalam variabel `local_14`, lalu membandingkannya dengan nilai `0x539`. Instruksi perbandingan (`CMP`) digunakan untuk mengecek apakah kedua nilai tersebut sama. Jika sama, maka akan menetapkan '*zero flag*' menjadi benar. Setelah itu, ada instruksi `JNZ` (*Jump if Not Zero*) yang akan mengarahkan program untuk melompat ke alamat tertentu jika '*zero flag*' tidak aktif. Jika '*zero flag*' aktif, maka program akan melanjutkan ke instruksi selanjutnya. Namun, karena `0xdeadbeef` berbeda dari `0x539`, '*zero flag*' tidak akan aktif dan instruksi `JNZ` akan menyebabkan program melompat ke alamat lain, yaitu ke akhir fungsi.

![Akhir Fungsi getFlag()](/assets/img/posts/frida-labs-challenge-0xb/10.png)
_Akhir Fungsi getFlag()_

Kita ingin mencegah aplikasi mengambil lompatan ini agar bisa melanjutkan ke instruksi berikutnya, yang mendekode dan mencatat *flag*. Salah satu cara untuk melakukan ini adalah dengan memodifikasi instruksi `JNZ` menjadi instruksi `NOP` (*No Operation*). Instruksi `NOP` tidak melakukan apapun kecuali meneruskan kontrol ke instruksi berikutnya. Jadi, dengan mengganti `JNZ` dengan `NOP`, program akan melanjutkan eksekusi secara linier dan memungkinkan pencatatan *flag*. Sebagai alternatif, kita bisa mencoba menggunakan instruksi seperti `JE` (*Jump if Equal*), yang berfungsi kebalikan dari `JNZ`.

Sekarang kita telah memahami strateginya, mari kita pelajari bagaimana menggunakan `X86Writer` untuk mengimplementasikan solusi kita.

Kita akan mulai dengan mencari alamat dari instruksi `JNZ` yang ingin kita *patch*.

![Offset Pemeriksaan IF](/assets/img/posts/frida-labs-challenge-0xb/11.png)
_Offset Pemeriksaan IF_

Untuk menemukan alamat ini, kita perlu menghitung *offset* dengan mengurangi nilai `0x20e2a` dari *base address* `0x00010000` (x86). Kemudian, kita gunakan *offset* ini bersama *base address* untuk menemukan alamat yang sebenarnya di memori.

Berikut adalah cara menghitung alamat sebenarnya dalam kode:

```bash
Module.getBaseAddress("libfrida0xb.so")
# "0xc06c0000"

Module.getBaseAddress("libfrida0xb.so").add(0x20e2a - 0x00010000)
# "0xc06d0e2a"
```
{: .nolineno }

Sekarang, kita sesuaikan *script* untuk memodifikasi instruksi:

```javascript
var jnz = Module.getBaseAddress("libfrida0xb.so").add(0x20e2a - 0x00010000);
var writer = new x86Writer(jnz);

try {

  writer.flush();

} finally {

  writer.dispose();
}
```

Untuk mengganti instruksi `JNZ` dengan `NOP`, kita akan menggunakan metode `putNop()` yang disediakan oleh `x86Writer`. 

![Dokumentasi putNop()](/assets/img/posts/frida-labs-challenge-0xb/12.png)
_Dokumentasi putNop()_

> Dokumentasi `x86Writer` bisa kamu baca lebih lanjut [di sini](https://frida.re/docs/javascript-api/#x86writer).
{: .prompt-tip }

Sebelum kita memperbarui *script* kita, mari pertimbangkan ini: berapa banyak instruksi `NOP` yang perlu kita sisipkan?

Instruksi `NOP` pada arsitektur x86 biasanya berukuran 1 byte. Namun, instruksi `JNZ` yang ingin kita ganti memiliki ukuran 6 byte. Kita bisa mengetahui ukuran instruksi ini menggunakan alat seperti Ghidra. Kita perlu menempatkan enam instruksi `NOP` untuk menutupi 6 byte dari instruksi `JNZ` yang asli. Ini penting untuk memastikan bahwa kita tidak mengganggu aliran kode selanjutnya dengan meninggalkan ruang kosong atau menghapus terlalu banyak kode.

```nasm
00020e2a 0f  85  d8       JNZ        LAB_00020f08
		 00  00  00
00020e30 8b  5d  dc       MOV        EBX ,dword ptr [EBP  + local_28 ]
```

Berikut adalah cara kita memperbarui *script* dengan memasukkan enam instruksi `NOP`:

```javascript
var jnz = Module.getBaseAddress("libfrida0xb.so").add(0x20e2a - 0x00010000);
var writer = new X86Writer(jnz);

try {
   
  writer.putNop()
  writer.putNop()
  writer.putNop()
  writer.putNop()
  writer.putNop()
  writer.putNop()

  writer.flush();

} finally {

  writer.dispose();
}
```

Mari kita jalankan *script*-nya.

```bash
➜ frida -U -f com.ad2001.frida0xb
```
{: .nolineno }

![Crash Access Protection](/assets/img/posts/frida-labs-challenge-0xb/13.png)
_Crash Access Protection_

Yah, kita mendapati *crash* karena melanggar proteksi akses. Ini terjadi karena bagian `.text` dari binary, tempat instruksi `JNZ` berada, biasanya tidak memiliki hak akses *write*. 

![Hak Akses .text Di Memori](/assets/img/posts/frida-labs-challenge-0xb/14.png)
_Hak Akses .text Di Memori_

Untuk mengatasi masalah ini, kita harus mengubah hak akses untuk bagian kode tersebut menggunakan fungsi `Memory.protect`. Fungsi ini memungkinkan kita untuk mengubah atribut perlindungan dari wilayah memori tertentu.

```javascript
Memory.protect(address, size, protection);
```

- `address`: Alamat awal dari wilayah memori yang perlindungannya ingin diubah.
- `size`: Ukuran dari wilayah memori yang diubah perlindungannya, diukur dalam byte.
- `protection`: Atribut perlindungan baru untuk wilayah memori tersebut, seperti izin *read*, *write*, atau *execute*.

Kita akan mengatur hak akses memori menjadi *read*, *wrute*, dan *execute* (`rwx`) untuk memastikan kita dapat menulis instruksi `NOP` tanpa *crash*. Berikut adalah cara menggunakannya:

```javascript
var jnz = Module.getBaseAddress("libfrida0xb.so").add(0x20e2a - 0x00010000);
Memory.protect(jnz, 0x1000, "rwx");
var writer = new X86Writer(jnz);

try {
   
  writer.putNop()
  writer.putNop()
  writer.putNop()
  writer.putNop()
  writer.putNop()
  writer.putNop()

  writer.flush();

} finally {

  writer.dispose();
}
```

Sekarang mari kita coba jalankan *script*-nya kembali.

```bash
➜ frida -U -f com.ad2001.frida0xb
```
{: .nolineno }

Klik tombol pada aplikasi dan cek log-nya.

![Berhasil Mendapatkan Flag Di x86](/assets/img/posts/frida-labs-challenge-0xb/15.png)
_Berhasil Mendapatkan Flag Di x86_

Wiih.. Kita mendapatkan *flag*. Jadi *patch*-nya berhasil!

## 0x4 - Patching using ARM64Writer

Sekarang, mari kita coba teknik modifikasi pada perangkat ARM64 menggunakan `ARM64Writer`. Pertama, kita akan dekompilasi *library* `libfrida0xb.so` untuk ARM64 menggunakan Ghidra.

![Ghidra ARM64](/assets/img/posts/frida-labs-challenge-0xb/16.png)
_Ghidra ARM64_

Kita dapat melihat *disassembly* ARM64 di sini.

> Karena fokus kita adalah pada *patching* menggunakan Frida dan bukan pada *reverse engineering* secara mendalam, kita akan langsung menuju ke bagian yang relevan.
{: .prompt-info }

```nasm
00115244 08  e5  14  71    subs       w8,w8,#0x539
00115248 21  07  00  54    b.ne       LAB_0011532c
0011524c 01  00  00  14    b          LAB_00115250
```

Berbeda dengan x86 yang memakai `cmp` untuk perbandingan, ARM64 memakai `subs` untuk mengurangi dan memeriksa kondisi. Instruksi `b.ne` (*branch if not equal*) akan melompat jika kondisinya tidak sesuai. Kita ingin menghindari lompatan ini dengan mengganti `b.ne` dengan instruksi `b`, yang selalu menjalankan cabang tanpa memeriksa kondisi, efektif menjadikannya 'selalu benar' (`if(true)`). Dengan demikian, eksekusi program akan melanjutkan ke instruksi selanjutnya tanpa melompat

![Disassembly ARM64](/assets/img/posts/frida-labs-challenge-0xb/17.png)
_Disassembly ARM64_

> Dokumentasi `ARM64Writer` bisa kamu baca lebih lanjut [di sini](https://frida.re/docs/javascript-api/#arm64writer).
{: .prompt-tip }

![Dokumentasi putBImm()](/assets/img/posts/frida-labs-challenge-0xb/18.png)
_Dokumentasi putBImm()_

Proses selanjutnya mirip dengan sebelumnya. Kita akan mencari alamat instruksi `b.ne` dan alamat instruksi berikutnya menggunakan Ghidra, lalu memodifikasinya dengan *script* berikut:

```javascript
var adr = Module.findBaseAddress("libfrida0xb.so").add(0x15248);  //Addres of the b.ne instruction (0x00115248- 0x00100000)
Memory.protect(adr, 0x1000, "rwx");
var writer = new Arm64Writer(adr);  //ARM64 writer object
var target = Module.findBaseAddress("libfrida0xb.so").add(0x1524c);  //Address of the next instruction  b  LAB_00115250 (0x0011524c- 0x00100000)

try {

  writer.putBImm(target);   // Inserts the <b target> instruction in the place of b.ne instruction

  
  console.log(`Branch instruction inserted at ${adr}`);
} finally {

  writer.dispose();
}
```

> Dalam arsitektur ARM64, kita tidak perlu khawatir tentang penyelarasan instruksi karena semuanya sudah diselaraskan dalam kelipatan 4 byte.
{: .prompt-tip }

Setelah menyiapkan *script*, kita jalankan dan cek apakah kita berhasil mendapatkan *flag*.

![Menjalankan Script Frida](/assets/img/posts/frida-labs-challenge-0xb/19.png)
_Menjalankan Script Frida_

Kemudian kita klik tombol di aplikasi dan periksa log untuk melihat hasilnya.

![Mendapatkan Flag](/assets/img/posts/frida-labs-challenge-0xb/20.png)
_Mendapatkan Flag_

Seperti yang diharapkan, kita berhasil mendapatkan flag! Jadi, proses *patching* telah berhasil.