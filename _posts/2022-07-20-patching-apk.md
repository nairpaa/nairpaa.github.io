---
title: 'Decompile & Patching APK'
date: 2022-07-20 10:10:22 +0700
categories: ['Android Hacking', 'Reverse Engineering']
tags: [android, integrity check, reverse engineering]     # TAG names should always be lowercase
author: nairpaa
---

*Decompile* adalah proses menggembalikan *source code* dari kode yang sudah di-*compile*. Setelah mendapatkan *source code*, kita dapat melakukan perubahan pada APK dan melakukan *recompile*, ini lah yang dinamakan *patching* APK.

Kegiatan *patching* APK ini biasanya digunakan untuk mem-*bypass* root dan SSL Pinning, dengan cara mengubah nilai dari kode yang dijadikan sebagai acuan pemeriksaan.

## 1. Scenario

Pada aplikasi `diva-beta.apk` terdapat soal **Harcoding Issues - Part 1** yang mana kita perlu menginputkan *vendor key* yang valid.

*Vendor key* yang valid pada soal ini adalah "**vendorsecretkey**", hal ini bisa diketahui karena *key* tersebut disimpan di *source code*-nya.

Ketika kita menginputkan *key* yang salah, akan muncul pesan:

> Access denied! See you in hell :D
{: .prompt-danger }

![DIVA: Jika salah input vendor key](/assets/img/posts/patching-apk/5-1.png)
  
Ketika kita menginputkan *key* yang valid, akan muncul pesan:

> Access granted! See you on the other side :)
{: .prompt-tip }

![DIVA: Jika input vendor key yang valid](/assets/img/posts/patching-apk/5-2.png)

Pada kesempatan kali ini, kita akan mempelajari tentang *patching* atau modifikasi APK. Kita akan mengubah nilai *key* yang valid sesuai yang kita inginkan.

## 2. Requirements

  
1. Device atau emulator Android
  - Salah satu emulator Android yang saya rekomendasikan adalah [Genymotion](https://www.genymotion.com/download/).
2. Java
  - [Java](https://www.java.com/en/download/help/windows_manual_download.html) adalah salah satu bahasa pemrogramman yang sering digunakan untuk membuat APK, dan beberapa tools yang akan kita gunakan juga menggunakan bahasa Java.
  - `❯ apt-get install default-jre && apt-get install default-jdk`
3. APK Studio
  - [APKStudio](https://github.com/vaibhavpandeyvpz/apkstudio/releases) adalah IDE *open source* berbasis Qt yang digunakan untuk membantu proses *reverse engineering* aplikasi Android.
4. Apktool
  - [Apktool](https://github.com/iBotPeaches/Apktool/releases) adalah alat yang digunakan untuk melakukan *decompile* dan *recompile* APK.
5. Android Debug Bridge
  - [ADB](https://developer.android.com/studio/command-line/adb) adalah *command line tool* serbaguna yang memungkinkan kita berkomunikasi dengan perangkat Android.
  - `❯ apt install android-tools-adb`
  - Install [APK](https://github.com/tjunxiang92/Android-Vulnerabilities/raw/master/diva-beta.apk) yang akan digunakan sebagai bahan praktik.
  - `❯ adb install diva-beta.apk`
6. Jadx
  - [Jadx](https://github.com/skylot/jadx/releases/download/v1.2.0/jadx-1.2.0.zip) adalah alat bantu untuk melakukan konversi dari Dex ke Java agar lebih mudah dibaca.
7. Uber APK Signer
  - [Uber APK Signer](https://github.com/patrickfav/uber-apk-signer/releases) adalah alat bantu untuk melakukan *signing* APK.

## 3. Setting up APKStudio

Jalankan APKStudio, pada menu **Settings** terdapat konfigurasi PATH dari masing-masing tools. Silahkan sesuaikan dengan kondisi Anda.

![Menyiapkan APKStuido](/assets/img/posts/patching-apk/1.png)

Keretangan:

- [AAPT2](https://developer.android.com/studio/command-line/aapt2?hl=id) (*Android Asset Packaging Tool*) adalah alat yang digunakan oleh Android Studio dan Plugin Android Gradle untuk mengompilasi dan memaketkan *resource* aplikasi.

## 4. Decompile APK

Untuk melakukan *decompile* APK, tekan tombol **Open APK** dan pilih `diva-beta.apk` yang akan kita gunakan sebagai bahan praktik.

![Decompile APK](/assets/img/posts/patching-apk/2.png)

Selanjutnya akan ada opsi seperti gambar di bawah:

![Decompile APK](/assets/img/posts/patching-apk/3.png)

Keterangan:
- **Output**: lokasi folder untuk menyimpan hasil *decompile*.
- **Decompile smali?**: untuk mendapatkan *.smali* (`apktool`).
- **Decompile resource?**: untuk mendapatkan [*resource*](https://www.codepolitan.com/memahami-app-resource-di-android-598c2b837f4fa) (`apktool`).
- **Decompile java?**: untuk mendapatkan *.java* (`jadx`).

Hasil dari *decompile*-nya akan terlihat seperti berikut:

![Hasil Decompile](/assets/img/posts/patching-apk/4-1.png)

Jika Anda ingin *decompile* APK secara manual gunakan perintah berikut:

```bash
# decompile .apk
❯ java -Xmx256m -jar apktool_2.5.0.jar d -o /apk/diva-beta.apk-decompiled /apk/diva-beta.apk

# konversi .dex ke .java
❯ jadx -r -d /apk/diva-beta.apk-decompiled /apk/diva-beta.apk
```

### 4.1. Smali vs Java

**[Smali](https://formasiberita.blogspot.com/2019/04/mengenal-istilah-smali-dan-baksmali.html)** adalah sebuah bahasa hasil dekompilasi file berekstensi `.dex` yang terdapat pada file `.apk`. Untuk melakukan *patching*, kita perlu untuk mengubah isi file `.smali`, lalu di-*recompile* menggunakan `apktool`.

Untuk mempermudah memahami `.smali`, kita bisa membandingkannya terlebih dahulu dengan file `.java` (yang dihasilkan oleh `jadx`).

Sebagai contoh pada aplikasi `diva-beta.apk` terdapat *class* `HardcodeActivity` yang menyimpan *vendor key*.

![HardcodeActivity.smali](/assets/img/posts/patching-apk/4-2.png)

Untuk lebih memahami kode tersebut, sebaiknya kita lihat isi file `.java`.

![HardcodeActivity.java](/assets/img/posts/patching-apk/4-3.png)

Terlihat pada gambar di atas, terdapat pemeriksaan string yang diinputkan. String atau *key* yang valid adalah ***vendorsecretkey***.

> Untuk memahami lebih lanjut tentang `.smali` dan *reverse engineering* APK Android, Anda bisa membaca artikel berikut:
- [Editing Smali](https://yohan.es/security/android/editing-smali/)
- [What's the best way to learn Smali?](https://stackoverflow.com/questions/5656804/whats-the-best-way-to-learn-smali-and-how-when-to-use-dalvik-vm-opcodes)
- [Reverse Engineering APK Android](https://yohan.es/security/android/)
{: .prompt-tip }

## 5. Patching APK

Setelah mengetahui bahwa terdapat *key* yang disimpan di *source code*, kita akan mengubah nilai *key* tersebut menjadi "***BelajarPatchingAPK1337***".

Untuk melakukannya, cukup mengubah isi file `.smali` menjadi seperti berikut:

![Mengubah nilai vendor key pada file .smali](/assets/img/posts/patching-apk/6.png)

Jangan lupa untuk menyimpan perubahan (`CTRL`+`S`) dan *recompile*, seperti berikut:

![Recompile APK](/assets/img/posts/patching-apk/7.png) 

Jika Anda ingin *recompile* APK secara manual gunakan perintah berikut:

```bash
java -Xmx256m -jar apktool_2.5.0.jar b /apk/diva-beta.apk-decompiled --use-aapt2
```

Hasil *recompile* APK akan tersedia pada folder `/dist/`, seperti berikut:

![Result Recompile APK](/assets/img/posts/patching-apk/8.png)

Kita akan mendapatkan pesan kesalahan ketika ingin menginstall APK yang telah kita *patching* sebelumnya, seperti berikut:

```bash
❯ adb install diva-beta.apk
Performing Streamed Install          
adb: failed to install diva-beta.apk: Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: Failed to collect certificates from /data/app/vmdl749566934.tmp/base.apk: Attempt to get length of null array]
```

Hal ini karena tidak ada sertifikat pada APK tersebut. Maka dari itu, kita akan menambahkan sertifikat baru menggunakan `Uber-Apk-Signer`.

## 6. Adding Certificates to APK

Pertama-tama, kita buat terlebih dahulu sertifikatnya menggunakan `keytool`, seperti berikut:

```bash
❯ keytool -genkey -v -keystore tryn3wbye.key -alias tryn3wbye -keyalg RSA -keysize 2048 -validity 365
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter keystore password:  
Re-enter new password:
What is your first and last name?
  [Unknown]:  n3wbye
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  Jakarta
What is the name of your State or Province?
  [Unknown]:  Jateng
What is the two-letter country code for this unit?
  [Unknown]:  ID
Is CN=n3wbye, OU=Unknown, O=Unknown, L=Jakarta, ST=Jateng, C=ID correct?
  [no]:  yes

Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 365 days
        for: CN=n3wbye, OU=Unknown, O=Unknown, L=Jakarta, ST=Jateng, C=ID
[Storing tryn3wbye.key]
```

Keterangan:
- `Keytool`: alat bantu untuk mengelola sertifikat.
- `-genkey`: membuat atau menambah data ke *keystore*.
- `-v`: mode *verbose*.
- `-keystore`: lokasi *keystore*.
- `-alias`: sebagai nama entri untuk diproses.
- `-keysize`: ukuran *key* dalam bit.
- `-validity`: validitas jumlah hari.

Setelah sertifikat baru berhasil dibuat, kita akan melakukan konfigurasi untuk *signing* pada APKStudio, seperti berikut:

![Setting Sertifikat](/assets/img/posts/patching-apk/9.png) 

Selanjutnya, pada APKStudio pilih menu *Project > Sign / Export* untuk menambahkan sertifikasi ke dalam APK.

![Menambahkan Sertifikat pada APK](/assets/img/posts/patching-apk/10.png) 

Jika Anda ingin *signing* APK secara manual gunakan perintah berikut:

```bash
java -Xmx256m -jar uber-apk-signer-1.2.1.jar -a /apk/diva-beta.apk-decompiled/dist/diva-beta.apk --allowResign --overwrite --ks <namacert>.key --ksPass <password> --ksAlias <namacert> --ksKeyPass <password>
```

## 7. Install and try the App

Sekarang APK dapat diinstall dan digunakan.

```bash
❯ adb install diva-beta.apk
Performing Streamed Install
Success
```

Terlihat pada gambar di bawah ini, ketika kita mencoba menggunakan *key* lama, akan muncul pesan:

> Access denied! See you in hell :D
{: .prompt-danger }

![DIVA-Patching: Menggunakan key lama](/assets/img/posts/patching-apk/11-1.png)

Dan jika kita menginputkan ***BelajarPatchingAPK1337***, akan muncul pesan:

> Access granted! See you on the other side :)
{: .prompt-tip }

![DIVA-Patching: Menggunakan key baru](/assets/img/posts/patching-apk/11-2.png)

---

Kita telah berhasil melakukan *decompile* dan *patching* APK. Semoga artikel ini bermanfaat untuk teman-teman.

**Happy Hacking!**