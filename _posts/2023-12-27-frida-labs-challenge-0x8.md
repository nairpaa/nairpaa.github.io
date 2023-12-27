---
title: '[Frida Labs] 08 - Introduction to Native Hooking'
date: 2023-12-27 14:44:22 +0700
categories: ['Mobile Application', 'Android Hacking']
tags: ['android', 'frida', 'ctf', 'ctf android', 'reverse engineering', 'frida-labs']     # TAG names should always be lowercase
author: nairpaa
---

Dalam kesempatan ini, kita akan mempelajari cara meng-*hook* fungsi *native* menggunakan Frida. 

Sebagai studi kasus, kita akan menerapkannya pada aplikasi **Challenge 0x8.apk**, yang bisa diunduh dari [sini](https://raw.githubusercontent.com/DERE-ad2001/Frida-Labs/main/Frida%200x8/Challenge%200x8.apk).

## 0x1 - Prerequisites

- Dasar *reverse engineering* menggunakan [JADX](https://github.com/skylot/jadx).
- Kemampuan untuk memahami kode Java.
- Kemampuan untuk menulis kode Javascript.
- Familiar dengan [ADB](https://developer.android.com/tools/adb).
- Perangkat Android yang sudah di-root.
- Dasar *assembly* dan *reversing* x64/ARM64

## 0x2 - Challenge 0x8

Tantangan kali ini akan berbeda dengan tantangan [sebelumnya](/tags/frida-labs/). Kali ini kita berurusan dengan *native library*. [Android NDK (Native Development Kit) ](https://developer.android.com/ndk) memungkinkan *developer* untuk menambahkan kode *native*, yang ditulis menggunakan bahasa seperti C dan C++, ke dalam aplikasi Android.

*Native code* ini dikompilasi menjadi *library* atau *shared objects* (`.so`), yang memungkinkan optimasi performa pada komponen-komponen kritis serta memberikan kontrol manajemen memori yang lebih baik kepada *developer*. 

Dalam skenario ini, kita akan menggunakan Frida untuk melakukan *hooking* ke *native library*.

Mari kita jalankan aplikasinya.

![Aplikasi Challenge 0x8](/assets/img/posts/frida-labs-challenge-0x8/1.png){: w="300" }
_Aplikasi Challenge 0x8_

Terdapat `editText`, coba kita input-kan sesuatu.

![Mencoba Memasukkan Teks](/assets/img/posts/frida-labs-challenge-0x8/2.png)
_Mencoba Memasukkan Teks_

Muncul pesan "**TRY AGAIN**". Mari kita dekompilasi menggunakan `JADX`.

![Class MainActivity](/assets/img/posts/frida-labs-challenge-0x8/3.png)
_Class MainActivity_

Kita melihat terdapat deklarasi fungsi *native* `cmpstr`.

```java
public native int cmpstr(String str);

static {
	System.loadLibrary("frida0x8");
}
```

Fungsi `cmpstr` ini mengambil string sebagai argumennya dan mengembalikan sebuah integer.

Aplikasi memuat *native library* `frida0x8` ke dalam memori. Mari kita amati proses yang terjadi pada fungsi *callback* dari tombol tersebut.

```java
button.setOnClickListener(new View.OnClickListener() { // from class: com.ad2001.frida0x8.MainActivity.1
    @Override // android.view.View.OnClickListener
    public void onClick(View v) {
        String ip = MainActivity.this.edt.getText().toString();
        int res = MainActivity.this.cmpstr(ip);
        if (res == 1) {
            Toast.makeText(MainActivity.this, "YEY YOU GOT THE FLAG " + ip, 1).show();
        } else {
            Toast.makeText(MainActivity.this, "TRY AGAIN", 1).show();
        }
    }
});
```

Kita dapat mengamati bahwa dalam fungsi *callback*, aplikasi memanggil metode `cmpstr` dengan teks dari `EditText` dan mengembalikan nilai integer. Aspek penting lainnya adalah bahwa input kita merupakan *flag* itu sendiri. Berikut ini adalah rangkuman dari apa yang telah kita ketahui:

- Aplikasi memuat *library* bernama `frida0x8` ke dalam memori secara dinamis saat *runtime*.
- Input pengguna merupakan *flag* yang dimaksud. Jadi kita harus meng-input-kan *flag* untuk mendapatkan pesan sukses.
- Input ini selanjutnya dikirimkan ke fungsi yang bernama `cmpstr` di dalam *library* `frida0x8`.
- Jika nilai yang dikembalikan adalah `1`, kita akan berhasil mendapatkan *flag*.

Sekarang, mari kita periksa *library* `frida0x8`.

Kita bisa melihat list *library* yang digunakan oleh aplikasi menggunakan `JADX` di *path* `/resources/lib/`.

![Daftar Library Aplikasi](/assets/img/posts/frida-labs-challenge-0x8/4.png)
_Daftar Library Aplikasi_

Di bagian ini, saya memilih menggunakan ARM64, mengingat ini adalah arsitektur yang paling umum pada perangkat Android fisik saat ini.

> Bagi yang menggunakan arsitektur lain, seperti x86 atau ARM, tahapan berikut tetap dapat dilanjutkan dengan melakukan beberapa penyesuaian.
{: .prompt-info }

Kita bisa mendapatkan file *library* ini menggunakan [apktool](https://apktool.org/).

```bash
➜ apktool d "Challenge 0x8.apk"
```
{: .nolineno }

![Dekompilasi APK Menggunakan Apktool](/assets/img/posts/frida-labs-challenge-0x8/5.png)
_Dekompilasi APK Menggunakan Apktool_

Dapat kita lihat, terdapat dua library di sini: `liblog.so` dan `libfrida0x8.so`. Namun, fokus kita hanya pada `frida0x8`, yang telah diubah namanya menjadi `libfrida0x8.so`. Sesuai konvensi, `lib` diberikan sebagai prefiks pada nama file *library*. Ekstensi `.so` menunjukkan bahwa ini adalah *shared objects*. 

Pastikan kamu memilih file *library* yang sesuai dengan arsitektur perangkat yang digunakan.

Untuk melakukan *reverse engineering* dan analisis pada file `libfrida0x8.so`, kita akan menggunakan alat yang bernama [Ghidra](https://ghidra-sre.org/). Kamu bisa menggunakan alat lain seperti [IDA](https://hex-rays.com/ida-pro/), [Radare](https://rada.re/n/), atau [Hopper](https://www.hopperapp.com/), namun saya memilih Ghidra karena merupakan alat gratis dan *open source*. Ghidra adalah sebuah *framework* untuk *software reverse engineering (SRE)* yang dikembangkan oleh [National Security Agency (NSA)](https://www.nsa.gov/). Ghidra menyediakan serangkaian *tools* dan kemampuan untuk menganalisis dan memahami fungsi dari biner yang telah dikompilasi.

> Jika ini adalah pengalaman pertama kamu menggunakan Ghidra, kamu bisa melihat video tutorialnya di [sini](https://www.youtube.com/watch?v=fTGTnrgjuGA). 
{: .prompt-tip }

![Import Library Pada Ghidra](/assets/img/posts/frida-labs-challenge-0x8/6.png)
_Import Library Pada Ghidra_

Mari kita klik `ok` dan lanjutkan prosesnya.

![Informasi Library Pada Ghidra](/assets/img/posts/frida-labs-challenge-0x8/7.png){: w="600" }
_Informasi Library Pada Ghidra_

Ghidra akan mengidentifikasi *signature* file. Mari kita lanjutkan dengan mengklik `ok`.

![Analisis Library](/assets/img/posts/frida-labs-challenge-0x8/8.png)
_Analisis Library_

Klik `Yes` dan tunggu hingga analisis selesai.

Setelah itu, navigasikan ke list *dropdown* **Functions** di sebelah kiri Ghidra.

![List Fungsi Pada Library](/assets/img/posts/frida-labs-challenge-0x8/9.png)
_List Fungsi Pada Library_

Kita bisa melihat fungsi `cmpstr`. Nama `Java_com_ad2001_frida0x8_MainActivity_cmpstr` yang tampak panjang hanyalah penambahan dari nama *package*.

Dalam Java, ketika mendeklarasikan metode *native*, kita menggunakan kata kunci `native` untuk menandakan bahwa implementasi metodenya diberikan dalam bahasa lain, umumnya C atau C++. Deklarasi metode *native* di kelas Java tidak menyertakan implementasi; ia hanya berfungsi sebagai *signature* yang menginformasikan *runtime* Java bahwa metode tersebut akan diimplementasikan dalam bahasa *native*. Konvensi penamaan metode ini mencakup nama *package* dan nama *class*.

Klik dua kali pada fungsi `cmpstr` untuk menampilkan disasembli dan dekompilasi dari fungsi tersebut.

![Disasembli Fungsi cmpstr](/assets/img/posts/frida-labs-challenge-0x8/10.png)
_Disasembli Fungsi cmpstr_

Panel kanan menunjukkan dekompilasi. Jika ini pertama kalinya kamu melakukan *reversing* pada *native library*, jangan khawatir, sebab semuanya akan menjadi lebih mudah seiring dengan bertambahnya latihan. 

Untuk membantu penjelasan, saya akan menyediakan *source code* dari fungsi *native* tersebut.

```c
#include <jni.h>
#include <string.h>
#include <cstdio>
#include <android/log.h>

extern "C"
JNIEXPORT jint JNICALL
Java_com_ad2001_frida0x8_MainActivity_cmpstr(JNIEnv *env, jobject thiz, jstring str) {
    const char *inputStr = env->GetStringUTFChars(str, 0);
    const char *hardcoded = "GSJEB|OBUJWF`MBOE~";
    char password[100];

    for (int i = 0; i < strlen(hardcoded) ; i++) {

        password[i] = (char)(hardcoded[i] - 1);
    }

    password[strlen(hardcoded)] = '\0';
    int result = strcmp(inputStr, password);
    __android_log_print(ANDROID_LOG_DEBUG, "input ", "%s",inputStr);
    __android_log_print(ANDROID_LOG_DEBUG, "Password", "%s",password);
    env->ReleaseStringUTFChars(str, inputStr);

    // Returning result: 1 if equal, 0 if not equal
    return (result == 0) ? 1 : 0;
}


```

Mari saya jelaskan secara singkat tentang kode ini.

```c
extern "C" JNIEXPORT jint JNICALL
Java_com_ad2001_frida0x8_MainActivity_cmpstr(JNIEnv *env, jobject thiz, jstring str)
```

- Kode ini mendeklarasikan sebuah fungsi [JNI (Java Native Interface)](https://en.wikipedia.org/wiki/Java_Native_Interface) yang bernama `cmpstr`.
- Fungsi ini dirancang untuk dipanggil dari kode Java dengan nama `Java_com_ad2001_frida0x8_MainActivity_cmpstr`.
- Fungsi ini menerima tiga parameter: `env` yang merujuk pada *environment* JNI, `thiz` sebagai objek Java, dan `str` sebagai string Java.

```c
const char *inputStr = env->GetStringUTFChars(str, 0);
```

- Kode ini mengambil string masukan dari Java (`jstring`) dan mengonversinya menjadi string gaya C (`const char*`). Ini memungkinkan string Java yang diberikan sebagai parameter untuk digunakan dalam fungsi *native* yang ditulis dalam C.

```c
const char *hardcoded = "GSJEB|OBUJWF`MBOE~";
char password[100];
for (int i = 0; i < strlen(hardcoded); i++) {
    password[i] = (char)(hardcoded[i] - 1);
}
```

- Variabel `hardcoded` berisi sebuah string yang telah dikodekan sebelumnya. Disamping itu, sebuah array `password` dideklarasikan untuk menyimpan hasil dekoding.
- Melalui *loop*, kode ini mengubah setiap karakter (ASCII) dalam `hardcoded` dengan menguranginya sebanyak 1, lalu menyimpan hasil perubahan tersebut dalam array `password`.

```c
int result = strcmp(inputStr, password);
```

- Kode ini menggunakan fungsi `strcmp` untuk membandingkan string input pengguna (`inputStr`) dengan string `password` yang telah disesuaikan.
- Hasil perbandingan disimpan dalam variabel `result`. Jika kedua string identik, [`strcmp`](https://www.programiz.com/c-programming/library-function/string.h/strcmp) akan mengembalikan 0, yang kemudian disimpan dalam `result`.

```c
env->ReleaseStringUTFChars(str, inputStr);
```

- Kode ini melepaskan *resource* yang terkait dengan string input (`inputStr`). Ini merupakan bagian penting dari pengelolaan memori dalam JNI, memastikan bahwa memori yang dialokasikan untuk string C-*style* yang dihasilkan dari string Java dilepaskan setelah tidak lagi dibutuhkan.

```c
return (result == 0) ? 1 : 0;
```

- Kode ini menggunakan operator kondisional untuk mengembalikan nilai 1 jika string input pengguna (`inputStr`) dan string `password` sama (dengan kata lain, jika `strcmp` mengembalikan 0, menandakan kesamaan), dan mengembalikan 0 jika keduanya tidak sama.

## 0x3 - Hooking the native functions

Kita bisa menyelesaikan tantangan ini dengan meng-*hook* fungsi `strcmp` dan men-*dumping* argumen-argumennya.

Untuk meng-*hook* fungsi *native*, kita bisa menggunakan API `Interceptor`. Berikut *template script*-nya:

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

- `Interceptor.attach`: Fungsi ini meng-*attach* *callback* pada alamat fungsi tertentu. `targetAddress` adalah alamat dari fungsi *native* yang ingin di-*hook*.
- `onEnter`: *Callback* ini dijalankan saat memasuki fungsi target. Ia memberikan akses ke argumen fungsi (`args`).
- `onLeave`: *Callback* ini dijalankan saat hendak meninggalkan fungsi target. Ia memberikan akses ke nilai yang dikembalikan oleh fungsi (`retval`).

Sekarang, pertanyaan yang muncul adalah bagaimana kita bisa mendapatkan alamat dari sebuah fungsi tertentu menggunakan Frida?

Terdapat berbagai cara untuk mencapainya, dan berikut adalah beberapa API Frida yang bisa kita gunakan:
1. `Module.enumerateExports()`: Untuk mendapatkan daftar semua *exports* yang tersedia.
2. `Module.enumerateImports()`: Untuk mendapatkan daftar semua *imports* pada modul.
3. `Module.getExportByName()`: Untuk mendapatkan alamat fungsi atau variabel yang di-*exports* berdasarkan namanya.
4. `Module.findExportByName()`: Sama seperti `getExportByName`, digunakan untuk mencari *exports* berdasarkan nama.
5. Hitung *offset* dan gunakan `add()` pada alamat yang diperoleh dari `Module.getBaseAddress()`: Untuk menghitung dan menavigasi ke alamat fungsi.

Namun, sebelum lebih jauh, mari kita pahami apa itu *exports* dan *imports*:

- *Exports* adalah fungsi atau variabel yang disediakan oleh sebuah *library* untuk digunakan oleh kode eksternal, mirip dengan fungsi yang kita sering gunakan dalam bahasa pemrograman seperti Python atau C.
- *Imports* adalah fungsi atau variabel yang diimpor oleh aplikasi kita dari *library* lain, seperti `libc.so` untuk mengakses fungsi standar seperti `strcmp`.

Sekarang, mari kita jelajahi masing-masing API ini satu per satu dengan menjalankan Frida dan meng-*attach* ke aplikasi yang kita analisis.

```bash
➜ frida -U -f com.ad2001.frida0x8
```
{: .nolineno }

### A. Module.enumerateExports()

API ini digunakan untuk mengenumerasi semua *exports* (*symbols*) dari sebuah modul tertentu. Fungsi-fungsi yang diekspor ini digunakan oleh aplikasi kita di Java. Fungsi ini membutuhkan satu argumen yaitu nama modul (baik itu *shared library* atau *executable*) yang ingin kita enumerasi.

Kita bisa melihat *imports* dan *exports* di tab **symbol tree** di Ghidra.

![Symbol Tree](/assets/img/posts/frida-labs-challenge-0x8/11.png)
_Symbol Tree_

Mari kita coba mendapatkan semua *exports* dari `libfrida0x8.so`.

```bash
Module.enumerateExports("libfrida0x8.so")
```
{: .nolineno }

![Module enumerateExports](/assets/img/posts/frida-labs-challenge-0x8/12.png)
_Module enumerateExports_

Dari daftar tersebut, kita dapat melihat alamat, nama, dan tipe dari setiap *exports*. 

Sekarang, mari kita fokus untuk mendapatkan alamat dari fungsi `cmpstr`. Kita akan menggunakan indeks `0` dari hasil enumerasi dan mengakses *key* yang berisi alamat sebagai berikut:

```bash
Module.enumerateExports("libfrida0x8.so")[0]
Module.enumerateExports("libfrida0x8.so")[0]["address"]
```
{: .nolineno }

![Alamat cmpstr](/assets/img/posts/frida-labs-challenge-0x8/13.png)
_Alamat cmpstr_

Alamat yang kita dapatkan adalah `0x7bc19f6864`. 

> Penting untuk diingat bahwa kita tidak boleh mengasumsikan alamat ini tetap sama setiap waktu karena [ASLR (*Address Space Layout Randomization*)](https://en.wikipedia.org/wiki/Address_space_layout_randomization) yang secara default aktif di Android akan mengubah alamat ini setiap kali aplikasi dijalankan.
{: .prompt-danger }

### B. Module.getExportByName()

Fungsi `Module.getExportByName(moduleName, exportName)` mengambil alamat simbol yang diekspor dengan nama yang diberikan dari modul (*shared library*). Jika kamu tidak tahu di *library* mana simbol yang diekspor kamu berada, kamu dapat mengirim `null`. Mari gunakan ini untuk menemukan alamat `cmpstr` lagi.

```bash
Module.getExportByName("libfrida0x8.so", "Java_com_ad2001_frida0x8_MainActivity_cmpstr")
```
{: .nolineno }

![Module enumerateExports](/assets/img/posts/frida-labs-challenge-0x8/14.png)
_Module enumerateExports_

Seperti yang kita lihat alamatnya sama dengan yang kita dapatkan sebelumnya.

### C. Module.findExportByName()

Ini sama dengan `Module.getExportByName()`. Perbedaannya adalah `Module.getExportByName()` memunculkan pengecualian jika ekspor tidak ditemukan, sementara `Module.findExportByName()` mengembalikan `null` jika ekspor tidak ditemukan.

```bash
Module.findExportByName("libfrida0x8.so", "Java_com_ad2001_frida0x8_MainActivity_cmpstr")
```
{: .nolineno }

![Module findExportByName](/assets/img/posts/frida-labs-challenge-0x8/15.png)
_Module findExportByName_

### D. Module.getBaseAddress()

Ketika API di atas tidak bekerja, kita bisa beralih ke `Module.getBaseAddress()`. API ini memberikan *base address* dari modul yang diberikan. Sebagai contoh, mari kita temukan *base address* dari *library* `libfrida0x8.so` dengan menggunakan perintah berikut:

```bash
Module.getBaseAddress("libfrida0x8.so")
```
{: .nolineno }

![Module getBaseAddress](/assets/img/posts/frida-labs-challenge-0x8/16.png)
_Module getBaseAddress_

Dengan *base address* ini, kita bisa menentukan alamat fungsi tertentu dengan menambahkan *offset* yang sesuai. Untuk menemukan *offset*, salah satu cara yang bisa dilakukan adalah dengan menggunakan Ghidra, seperti yang ditunjukkan di bawah ini:

![Mencari Offset Fungsi Menggunakan Ghidra](/assets/img/posts/frida-labs-challenge-0x8/17.png)
_Mencari Offset Fungsi Menggunakan Ghidra_

Ditemukan bahwa *offset* untuk `cmpstr` adalah `0x864`. Karena Ghidra memuat biner dengan *base address* default `0x100000`, kita perlu menyesuaikan *offset* ini dengan mengurangkan *base address* yang diberikan oleh Ghidra untuk mendapatkan nilai *offset* sebenarnya.

Dengan menambahkan `0x864` ke *base address* `0x7bc19f6000` yang kita dapatkan dari `libfrida0x8.so`, kita bisa mencari alamat aktual dari `cmpstr`:

```bash
Module.getBaseAddress("libfrida0x8.so").add(0x864)
```
{: .nolineno }

![Menambahkan Offset Pada Alamat](/assets/img/posts/frida-labs-challenge-0x8/18.png)
_Menambahkan Offset Pada Alamat_

### E. Module.enumerateImports()

Selain `Module.enumerateExports()`, kita juga memiliki `Module.enumerateImports()` yang memberikan daftar semua impor dari modul tertentu. Mari gunakan ini untuk mendapatkan daftar impor dari `libfrida0x8.so`.

![Module enumerateImports](/assets/img/posts/frida-labs-challenge-0x8/19.png){: w="600" }
_Module enumerateImports_

Dari daftar tersebut, kita bisa melihat berbagai fungsi yang diimpor, termasuk `strcmp`. Sekarang, mari kita temukan alamatnya dengan menggunakan perintah berikut:

```bash
Module.enumerateImports("libfrida0x8.so")[4]['address']
```
{: .nolineno }

![Mencari Alamat strcmp](/assets/img/posts/frida-labs-challenge-0x8/20.png)
_Mencari Alamat strcmp_

## 0x4 - Solve the challenge

Baik, sekarang setelah kita telah mengenal berbagai API, mari kita lanjutkan dengan menyelesaikan tantangan ini.

Dari penelitian kita terhadap *source code* atau *pseudo-code*, kita memahami bahwa string dari `EditText` dibandingkan dengan string lain menggunakan fungsi `strcmp()`. Kita juga menyadari bahwa *flag* adalah input yang kita berikan. 

Dengan demikian, strategi yang paling masuk akal adalah meng-*hook* fungsi `strcmp` agar kita dapat melihat string apa yang dibandingkan dengan input kita. `strcmp` menerima dua argumen; kedua-duanya adalah pointer ke string yang ingin dibandingkan. Kita dapat mencoba untuk men-*dump* argumen-argumen ini untuk melihat apa yang terjadi saat fungsi ini dipanggil. 

> Penting untuk diingat bahwa arsitektur yang berbeda memiliki konvensi panggilan yang berbeda serta set register yang berbeda, jadi pastikan untuk memeriksa disasembli untuk memahami bagaimana argumen diserahkan pada `strcmp`.
>
> Berikut adalah tabel konvensi panggilan yang bisa kita rujuk, diambil dari [syscall.sh](https://syscall.sh/):
> 
> ![Call Convention Table](https://github.com/DERE-ad2001/Frida-Labs/raw/main/Frida%200x8/Solution/images/22.png)
_Call Convention Table_
> 
> Tabel ini berguna untuk memahami bagaimana argumen disusun dan diserahkan ke fungsi pada arsitektur tertentu, sangat penting untuk memahami ini saat kita akan meng-*hook* dan memodifikasi perilaku fungsi di memori.
{: .prompt-tip }

Baiklah, sekarang mari kita mulai menulis *script* untuk meng-*hook* fungsi `strcmp`.

```javascript
Interceptor.attach(targetAddress, {
    onEnter: function (args) {
 
        // Modify or log arguments if needed
    },
    onLeave: function (retval) {
      
        // Modify or log return value if needed
    }
});
```

Pertama-tama, kita perlu menemukan alamat untuk `strcmp`. Kamu bisa menggunakan salah satu API yang telah kita pelajari sebelumnya. Saya akan memilih `Module.findExportByName()` karena kita tahu bahwa fungsi C seperti `strcmp` berada di *library* `libc.so` dan diekspor.

```bash
Module.findExportByName("libc.so", "strcmp");
# "0x7cbe727700"
```
{: .nolineno }

Sekarang kita akan menyimpan ini dalam variabel dan mengatur alamat targetnya.

```javascript
var strcmp_adr =  Module.findExportByName("libc.so", "strcmp");
Interceptor.attach(strcmp_adr, {
    onEnter: function (args) {
 
        // Modify or log arguments if needed
    },
    onLeave: function (retval) {
      
        // Modify or log return value if needed
    }
});
```

Selanjutnya, kita akan menambahkan `console.log` untuk memastikan semuanya berfungsi sebagaimana mestinya.

```javascript
var strcmp_adr =  Module.findExportByName("libc.so", "strcmp");
Interceptor.attach(strcmp_adr, {
    onEnter: function (args) {
 
        console.log("Hooking the strcmp function");
        
    },
    onLeave: function (retval) {
      
        // Modify or log return value if needed
    }
});
```

Sekarang kita akan menjalankan Frida dan menguji *script* ini.

![Mencoba Script Frida](/assets/img/posts/frida-labs-challenge-0x8/21.png)
_Mencoba Script Frida_

Selanjutnya, kita akan memicu fungsi `strcmp` dengan mengklik tombol di aplikasi.

![Trigger Script Frida Dengan Mengklik Tombol](/assets/img/posts/frida-labs-challenge-0x8/22.png)
_Trigger Script Frida Dengan Mengklik Tombol_

Ini berhasil tetapi mencetak pesan log berkali-kali. Ini terjadi karena kita telah meng-*hook* setiap panggilan `strcmp` di aplikasi, yang mungkin lebih dari yang kita inginkan. Mari kita perbaiki ini dengan memberikan filter tertentu.

Kita tahu bahwa salah satu string dalam `strcmp` adalah input kita. Jadi, kita dapat mengidentifikasi dan menangkapnya dengan string tertentu. Namun, kita belum tahu apakah input kita adalah argumen pertama atau kedua. Kita bisa menentukan ini dengan melihat dekompilasi di Ghidra atau melalui eksperimen.

Untuk membaca string dari memori dengan Frida, kita bisa menggunakan `Memory.readUtf8String()`, yang membaca string `utf` dari memori berdasarkan alamat yang diberikan. Karena `args` adalah array pointer yang berisi argumen untuk fungsi `strcmp`, kita dapat mengakses argumen pertama dengan `args[0]`.

```javascript
var strcmp_adr =  Module.findExportByName("libc.so", "strcmp");
Interceptor.attach(strcmp_adr, {
    onEnter: function (args) {
 
        var arg0 = Memory.readUtf8String(args[0]);

    },
    onLeave: function (retval) {
      
        // Modify or log return value if needed
    }
});
```

Kemudian, kita akan menambahkan kondisi `if` dan menggunakan metode `includes` untuk memfilter *hook* `strcmp`.

```javascript
var strcmp_adr =  Module.findExportByName("libc.so", "strcmp");
Interceptor.attach(strcmp_adr, {
    onEnter: function (args) {
 
        var arg0 = Memory.readUtf8String(args[0]);
        if(arg0.includes("Hello")){
        
            console.log("Hookin the strcmp function");
            
        }

    },
    onLeave: function (retval) {
      
        // Modify or log return value if needed
    }
});
```

Ini akan memeriksa apakah argumen pertama termasuk string "**Hello**". Jika ya, maka akan mencetak pesan log.

Setelah meng-*restart* Frida dan memasukkan "Hello" di `EditText` kita, kita akan memicu `strcmp` dengan mengklik tombol. 

> Ini penting: selalu *restart* Frida saat meng-*hook* fungsi *native*.
{: .prompt-warning }

![Mencoba Ulang Script Frida](/assets/img/posts/frida-labs-challenge-0x8/23.png)
_Mencoba Ulang Script Frida_

Kini, kita hanya melihat satu pesan log, yang merupakan perbaikan dari sebelumnya.

Akhirnya, kita akan mencetak argumen kedua, yaitu *flag*, yang dibandingkan dengan input kita.

```javascript
var strcmp_adr =  Module.findExportByName("libc.so", "strcmp");
Interceptor.attach(strcmp_adr, {
    onEnter: function (args) {

        var arg0 = Memory.readUtf8String(args[0]);   //first argument
        var flag = Memory.readUtf8String(args[1]);   //second argument

        if(arg0.includes("Hello")){ 
        
            console.log("Hookin the strcmp function");
            console.log("Input " + arg0);
            console.log("The flag is "+ flag);
            
        }

    },
    onLeave: function (retval) {
      
        // Modify or log return value if needed
    }
});
```

Mari kita jalankan.

![Berhasil Mendapatkan Flag](/assets/img/posts/frida-labs-challenge-0x8/24.png)
_Berhasil Mendapatkan Flag_

Yey.. Kita berhasil mendapatkan *flag*.

Kini, kita akan memasukkannya ke dalam aplikasi kita dan melihat hasilnya.

![Input Flag](/assets/img/posts/frida-labs-challenge-0x8/25.png)
_Input Flag_

Inilah cara kita bisa menggunakan Frida untuk meng-*hook* fungsi *native* dan ini akan sangat berguna dalam tantangan-tantangan [berikutnya](/tags/frida-labs/).