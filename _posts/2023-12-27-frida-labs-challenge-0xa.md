---
title: '[Frida Labs] 10 - Calling a Native Function'
date: 2023-12-27 23:47:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam bagian ini, kita akan mempelajari cara memanggil fungsi *native* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0xA.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200xA/Challenge%200xA.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.
- *Basics of x64/ARM64 assembly and reversing*.

## 0x2 - Challenge 0xA

Mari kita mulai dengan menginstal aplikasi.

![Aplikasi Challenge 0xA](/assets/img/posts/frida-labs-challenge-0xa/1.png)
_Aplikasi Challenge 0xA_

Seperti biasa, kita akan memulai dengan menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0xa/2.png)
_Class MainActivity_

APK ini mirip dengan sebelumnya. Dari analisis awal, kita melihat bahwa aplikasi memuat *library* `frida0xa` di bagian bawah.

```java
static {
	System.loadLibrary("frida0xa");
}
```

Kita juga menemukan adanya metode `JNI` di awal.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0xa/3.png)
_Metode JNI_

Metode ini mengembalikan string. Dalam aplikasi, metode dari *library* `frida0xa.so` ini dipanggil untuk mengatur teks di `TextView`, yang menampilkan teks "**Hello Hackers**". 

Selanjutnya, kita akan dekompilasi APK ini menggunakan apktool untuk memeriksa *library*-nya.

![Daftar Native Library](/assets/img/posts/frida-labs-challenge-0xa/4.png)
_Daftar Native Library_

Setelah itu, kita akan memuat *library* ini ke dalam Ghidra untuk analisis lebih lanjut.

Dan setelah selesai dianalisis, kita lihat fungsi yang tersedia.

![Daftar Fungsi Pada Library](/assets/img/posts/frida-labs-challenge-0xa/5.png)
_Daftar Fungsi Pada Library_

Kita menemukan dua fungsi:
- `get_flag()`
- `Java_com_ad2001_frida0xa_MainActivity_stringFromJNI()`

Kita tahu bahwa `TextView` memanggil `stringFromJNI()` untuk menampilkan "**Hello Hackers**". Kamu bisa memverifikasinya dengan melihat dekompilasi.

![Metode stringFromJNI()](/assets/img/posts/frida-labs-challenge-0xa/6.png)
_Metode stringFromJNI()_

Selanjutnya, ada metode `get_flag()`, yang tidak dideklarasikan di Java *space* dan juga tidak dipanggil dari mana pun di *library*. Kita dapat memverifikasinya dengan melihat *cross-reference*-nya, yang hanya terdapat di tabel [FDE (*Frame Description Entry*)](https://refspecs.linuxfoundation.org/LSB_3.0.0/LSB-Core-generic/LSB-Core-generic/ehframechpt.html).

![Melihat Cross-Reference get_flag()](/assets/img/posts/frida-labs-challenge-0xa/7.png)
_Melihat Cross-Reference get_flag()_

![Daftar Cross-Reference](/assets/img/posts/frida-labs-challenge-0xa/8.png)
_Daftar Cross-Reference_

Mari kita lihat dekompilasi metode ini.

![Metode get_flag()](/assets/img/posts/frida-labs-challenge-0xa/9.png){: w="600" }
_Metode get_flag()_

Fungsi ini mengambil dua nilai integer, menjumlahkannya, dan memeriksa apakah hasilnya sama dengan 3. Jika iya, maka ia mendekode string `FPE>9q8A>BK-)20A-#Y` dan menampilkan *flag*. 

Jadi, untuk mendapatkan *flag*, kita perlu memanggil metode ini. Mari kita gunakan Frida untuk melakukannya.

## 0x3 - Calling the native function

Ini sangat mudah dilakukan. Berikut adalah *template script*:

```javascript
var native_adr = new NativePointer(<address_of_the_native_function>);
const native_function = new NativeFunction(native_adr, '<return type>', ['argument_data_type']);
native_function(<arguments>);
```

Mari kita uraikan langkah demi langkah.

```javascript
var native_adr = new NativePointer(<address_of_the_native_function>);
```

Untuk memanggil fungsi *native* dengan Frida, kita membutuhkan objek `NativePointer` dengan alamat fungsi *native*. Setelah itu, kita membuat objek `NativeFunction`, yang menjadi JavaScript *wrapper* untuk fungsi *native*.

```javascript
NativeFunction(native_adr, '<return type>', ['argument_data_type']);
```

Argumen pertama haruslah objek `NativePointer`, argumen kedua adalah tipe pengembalian dari fungsi *native*, argumen ketiga adalah daftar dari tipe data argumen yang akan diberikan ke fungsi *native*. Sekarang kita dapat memanggil metode tersebut seperti yang kita lakukan di Java *space*.

```javascript
native_function(<arguments>);
```

Pertama, kita butuh alamat untuk `get_flag`. Kita bisa menemukan *offset* menggunakan Ghidra dan menambahkannya ke *base address* `libfrida0xa.so`.

```bash
➜ frida -U -f com.ad2001.frida0xa
```
{: .nolineno }

```bash
Module.getBaseAddress("libfrida0xa.so")
# "0x7bc194e000"
```
{: .nolineno }

![Alamat get_flag()](/assets/img/posts/frida-labs-challenge-0xa/10.png)
_Alamat get_flag()_

Dengan mengurangi alamat ini dari *base* di gambar (`0x0011dd60`), kita mendapatkan *offset*. Kita asumsikan bahwa Ghidra memuat binari pada `0x00010000`. Kita bisa menemukan basis ini di peta memori.

Kita dapat mendapatkan *offset* dengan mengurangi alamat ini dari *base* yang di gambar (`0x0011dd60`). Secara default, ghidra memuat binari pada `0x00010000`. Kita dapat menemukan basis ini di **Memory Map**.

![Alamat Basis Ghidra](/assets/img/posts/frida-labs-challenge-0xa/11.png)
_Alamat Basis Ghidra_

Dengan demikian, *offset* adalah `0x0011dd60` - `0x00100000` = `0x10dd60`. Sekarang kita bisa menambahkannya ke *base*.

```bash
Module.getBaseAddress("libfrida0xa.so") .add(0x1dd60)
# "0x7bc196bd60"
```
{: .nolineno }

Sekarang kita punya alamatnya.

```javascript
var adr = Module.findBaseAddress("libfrida0xa.so").add(0x1dd60)
```

Kemudian kita buat objek `NativePointer`.

```javascript
var adr = Module.findBaseAddress("libfrida0xa.so").add(0x1dd60) //Address of the get_flag() function
var get_flag_ptr = new NativePointer(adr);  
```

Sekarang kita buat objek `NativeFunction`. Argumen pertama adalah `get_flag_adr`. Karena `get_flag` adalah fungsi `void`, kita gunakan `void`. Fungsi `get_flag` memiliki dua argumen integer, jadi kita gunakan `['int', 'int']`.

```javascript
var adr = Module.findBaseAddress("libfrida0xa.so").add(0x1dd60) //Address of the get_flag() function
var get_flag_ptr = new NativePointer(adr); 
const get_flag = new NativeFunction(get_flag_ptr, 'void', ['int', 'int']);
```

Untuk melewati pengecekan `if`, kita perlu agar total penjumlahan argumen satu dan dua menjadi **3**. Oleh karena itu, kita akan memasukkan nilai **1** untuk argumen pertama dan nilai **2** untuk argumen kedua, sehingga penjumlahan keduanya menghasilkan total **3**.

```c
f (param_1 + param_2 == 3) {
local_30 = 0;
while( true ) {
  uVar1 = __strlen_chk("FPE>9q8A>BK-)20A-#Y",0xffffffff);
  if (uVar1 <= local_30) break;
  local_20[local_30] = "FPE>9q8A>BK-)20A-#Y"[local_30] + (char)(local_30 << 1);
  local_30 = local_30 + 1;
}
local_d = 0;
__android_log_print(3,&DAT_0001bf62,"Decrypted Flag: %s",local_20);
}
```

Jadi, *script* final akan terlihat seperti berikut:

```javascript
var adr = Module.findBaseAddress("libfrida0xa.so").add(0x1dd60) //Address of the get_flag() function
var get_flag_ptr = new NativePointer(adr); 
const get_flag = new NativeFunction(get_flag_ptr, 'void', ['int', 'int']);
get_flag(1,2);
```

Mari kita jalankan frida dan coba *script* ini dan periksa log-nya.

![Berhasil Mendapatkan Flag](/assets/img/posts/frida-labs-challenge-0xa/12.png)
_Berhasil Mendapatkan Flag_

Yay! Kita mendapatkan *flag*. Frida memang keren. :)