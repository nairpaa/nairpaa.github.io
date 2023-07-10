---
title: '0x2 - Cobalt Strike'
date: 2023-07-10 19:15:22 +0700
categories: ['Red Team', 'C2']
tags: [red team, introduction, fundamental, c2, cobalt strike]     # TAG names should always be lowercase
author: nairpaa
---

Pada bagian ini, kita akan mempelajari tentang dasar-dasar C2 Cobal Strike.

## 0x1 - Cobal Strike

Pada tahun 2012, [Raphael Mudge](https://www.cobaltstrike.com/blog/author/rsmudge/) membuat Cobalt Strike untuk melakukan pengujian keamanan. Cobalt Strike merupakan salah satu C2 *framework* publik pertama.

Pada tahun 2020, Cobalt Strike diakuisisi oleh HelpSystems LLC.

Cobalt Strike adalah *tool* komersial yang digunakan untuk operasi *Penetration Test* dan simulasi serangan *Advanced Persistent Threat (APT).* 

*Tool* ini memiliki berbagai kemampuan, seperti *spear phishing*, *exploitation*, *post-exploitation*, dan *C2 communication*.

## 0x3 - Starting the Team Server

Team Server adalah komponen pada Cobalt Strike yang berfungsi sebagai server pusat yang mengendalikan C2.

Untuk menjalankan Team Server ini, masuk ke direktori di mana Cobalt Strike server terinstal dan jalankan `teamserver` *executable*.

```bash
➜ sudo ./teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile
```

Keterangan:
- `10.10.5.50`: alamat IP dari server yang menjalankan Cobalt Strike server.
- `Passw0rd!`: *shared password* yang digunakan untuk mengakses Cobalt Strike server.
- `webbug.profile`: contoh profil Malleable C2 (akan dibahas lebih rinci nanti).

![Menjalankan Team Server](/assets/img/posts/cobalt-strike/1.png)
_Menjalankan Team Server_

> Disarankan untuk menggunakan Team Server menggunakan *session* tmux atau dijalankan sebagai *service* agar proses tidak terhenti ketika terminal terhenti.
{: .prompt-tip }

Setelah Team Server berhasil dijalankan, selanjutnya jalankan Cobalt Strike client. 

![Profile Cobalt Strike Client](/assets/img/posts/cobalt-strike/2.png)
_Profile Cobalt Strike Client_

Keterangan:
- `Alias`: nama profil koneksi untuk terhubung ke Team Server.
- `Host`: alamat IP dari Team Server.
- `Port`: port yang dijalankan oleh Team Server (default-nya `50050`).
- `User`: username yang digunakan untuk mengakses Team Server.
- `Password`: *shared password* yang dibuat pada saat menjalankan Team Server.

Saat terhubung ke Team Server pertama kalinya, Cobalt Strike client akan meminta konfirmasi *fingerprint* SHA256. Hal ini dilakukan untuk memastikan tidak ada yang menyadap *traffic* antara client dan server.

![Fingerprint SHA256](/assets/img/posts/cobalt-strike/3.png)
_Fingerprint SHA256_

Berikut adalah tampilan awal dari Cobal Strike:

![Tampilan Cobalt Strike](/assets/img/posts/cobalt-strike/4.png)
_Tampilan Cobalt Strike_

## 0x4 - Listener Management

Langkah selanjutnya adalah menyiapkan "*listener*", yang akan mendengarkan koneksi yang masuk dari *beacon*.

> **Beacon** adalah nama *payload* pada Cobalt Strike.
{: .prompt-info }

Ada dua jenis *listener* utama, yaitu: **egress** dan **peer-to-peer**.

### A. Egress Listeners

*Egress listener* adalah *listener* yang memungkinkan *beacon* berkomunikasi keluar jaringan target menuju Team Server.

Default dari *listener* jenis ini adalah HTTP/S dan DNS, di mana *beacon* akan mengenkapsulasi *traffic* C2 melalui masing-masing protokol.

Untuk mengelola *listener*, buka menu **Cobalt Strike > Listener** atau klik **ikon *headphone***.

![Listener Menu](/assets/img/posts/cobalt-strike/5.png)
_Listener Menu_

Setelah diklik akan muncul tab **Listener**, disini kita bisa menambah, mengubah, menghapus dan me-*restart* *listener*.

#### HTTP

HTTP *listener* memungkinkan *beacon* untuk mengirimkan dan menerima pesan C2 melalui *request* HTTP GET dan/atau POST.

Untuk membuat HTTP *listener*, klik **Add**. Pastikan **Beacon HTTP** dipilih di bagian jenis *payload* dan beri nama *listener*.

Pada bagian HTTP host secara default akan diisi dengan alamat IP dari Team Server, tetapi akan lebih realistis dengan menggunakan nama domain.

> Nama *listener* digunakan di seluruh perintah CLI Cobalt Strike, jadi pilihlah nama yang memungkinkan kita dengan mudah mengenalinya.
{: .prompt-tip }

![HTTP Listener](/assets/img/posts/cobalt-strike/6.png)
_HTTP Listener_

Ketika disimpan, Team Server akan menjalankan port 80.

```bash
➜ sudo ss -lntp
State            Recv-Q           Send-Q                       Local Address:Port                        Peer Address:Port           Process
LISTEN           0                50                                       *:80                                     *:*               users:(("TeamServerImage",pid=2172,fd=7))
```

#### DNS

DNS *listener* memungkinkan *beacon* untuk mengirimkan dan menerima pesan C2 melalui protokol DNS.

Ada beberapa jenis *record* DNS yang dapat digunakan untuk mengirimkan pesan C2, termasuk *record* `A`, `AAAA`, dan `TXT`.

*Record* `TXT` biasanya lebih sering digunakan karena dapat menampung data yang banyak, menjadikannya lebih efisien untuk mengirimkan pesan C2.

Namun, untuk menggunakan *listener* DNS, kita perlu membuat setidaknya satu *record* DNS yang membuat Team Server menjadi otoritatif untuk domain tersebut.

Pada umumnya, server DNS otoritatif didefinisikan dalam *record* `NS` (*Name Server*).

Berikut adalah contoh konfigurasi *record* DNS yang saya gunakan:

| Name   | Type | Data                 |
|--------|------|----------------------|
| @      | A    | 10.10.5.50           |
| ns1    | A    | 10.10.5.50           |
| pics   | NS   | ns1.nairpaa.me. |

![DNS Listener](/assets/img/posts/cobalt-strike/7.png)
_DNS Listener_

*Beacon* DNS dapat melakukan *lookup request*, seperti `<c2data>.pics.nairpaa.me`, yang kemudian akan dirutekan melalui infrastruktur DNS internet. 

Pada akhirnya *request* tersebut akan sampai ke Team Server, karena kita telah menyiapkan *records* yang sesuai untuk mengatakan bahwa server DNS **nairpaa.me** memiliki otoritas subdomain `pics`. 

Setelah *listener* disimpan, kita bisa menguji *records* dengan melakukan *lookup* sembarang. Default respons dari Team Server adalah `0.0.0.0`.

```bash
➜ dig @ns1.nairpaa.me test.pics.nairpaa.me +short
0.0.0.0
```

> **OPSEC**
> 
> Karena `0.0.0.0` adalah respons default (dan juga agak tidak masuk akal), Team Server Cobalt Strike dapat mudah diketahui. 
>
> Respons ini dapat diubah dalam profil Malleable C2.
{: .prompt-danger }

### B. Peer-to-peer

Berbeda dengan *engress listener*, P2P *listener* tidak berkomunikasi dengan Team Server secara langsung. Sebaliknya, P2P *listener* dirancang untuk melakukan *chaining* beberapa *beacon* bersama-sama dalam hubungan *parent/child*.

Alasan utama penggunaan *listener* ini adalah:
1. Untuk mengurangi jumlah host yang berkomunikasi ke Team Server. Karena semakin tinggi volume *traffic*, semakin besar kemungkinannya untuk terdeteksi.
2. Untuk menjalankan *beacon* pada komputer yang tidak ada akses ke luar jaringan.

Cobalt Strike memiliki dua jenis P2P *listener*, yaitu **Server Message Block (SMB)** dan **raw TCP**.

#### SMB

Konfigurasi SMB *listener* sangat sederhana karena hanya memiliki satu opsi, yaitu nama dari *pipename*.

Nama default-nya adalah `msagent_##` (di mana `##` adalah angka acak dalam heksadesimal), tetapi kita dapat menentukan nama apa pun yang diinginkan.

*Payload beacon* SMB akan memulai server *pipename* tersebut untuk mendengarkan koneksi yang masuk. *Pipename* ini dapat diakses secara lokal dan remote.


> **OPSEC**
> 
> Penamaan *pipe* default ini adalah tanda penggunaan Cobalt Strike. Strategi yang baik adalah meniru nama yang dikenal digunakan oleh aplikasi umum atau Windows itu sendiri. 
>
> Jalankan `PS C:> ls \\.\pipe\`, lalu buat *pipename* yang mirip dengan salah satu direktori yang ada.
{: .prompt-danger }

![SMB Listener](/assets/img/posts/cobalt-strike/8.png)
_SMB Listener_

#### TCP

*Beacon* TCP akan mengikat (*bind*) dan mendengarkan pada nomor port yang ditentukan.

Kita juga dapat menentukan apakah *beacon* akan mengikat hanya ke localhost (`127.0.0.1`) atau ke semua *interface* (`0.0.0.0`).

![TCP Listener](/assets/img/posts/cobalt-strike/9.png)
_TCP Listener_

Untuk mempelajari Cobalt Strike saya sarankan Anda untuk mencoba membuat berbagai jenis *listerner*.

Berikut adalah berbagai jenis *listener* (HTTP, DNS, SMB, dan TCP) yang bisa Anda coba:

![List Listener](/assets/img/posts/cobalt-strike/10.png)
_List Listener_

## 0x5 - Generating Payloads

Terdapat berbagai jenis *payload* yang dapat dihasilkan melalui menu **Payloads** dalam Cobalt Strike, diantaranya:

### A. HTML Application

Menghasilkan file `.hta` (biasanya dikirim melalui browser melalui teknik *social engineering*) yang menggunakan VBScript yang disematkan untuk menjalankan *payload*. Ini hanya menghasilkan *payload* untuk *listener egress* dan terbatas pada arsitektur x86.

### B. MS Office Macro

Menghasilkan kode VBA yang dapat ditambahkan ke dokumen Word atau Excel yang mengaktifkan macro. Ini hanya menghasilkan *payload* untuk *listener egress* tetapi kompatibel dengan Office x86 dan x64.

### C. Stager Payload Generator

Menghasilkan *stager payload* dalam berbagai bahasa termasuk C, C#, PowerShell, Python, dan VBA. Berguna saat membangun *payload* atau *custom exploit* Anda sendiri. Ini hanya menghasilkan *payload* untuk *listener egress*, tetapi mendukung x86 dan x64.

### D. Stageless Payload Generator

Sama seperti *stager payload*, tetapi menghasilkan *payload stageless* daripada *stager*. Memiliki format output yang lebih sedikit, misalnya tidak ada PowerShell, tetapi memiliki opsi tambahan untuk menentukan fungsi *exit* (*process* atau *thread*). Ini juga dapat menghasilkan *payload* untuk *listener* P2P.

### E. Windows Stager Payload

Menghasilkan *stager* yang telah dikompilasi sebelumnya sebagai EXE, *Service* EXE, atau DLL.

### F. Windows Stageless Payload

Menghasilkan *payload stageless* yang telah dikompilasi sebelumnya sebagai EXE, *Service* EXE, DLL, shellcode, serta PowerShell. Ini juga salah satu cara untuk menghasilkan *payload* untuk *listener* P2P.

### G. Windows Stageless Generate All Payloads

Menghasilkan setiap variasi *payload stageless*, untuk setiap *listener*, dalam x86 dan x64.

Berikut adalah contoh menghasilkan setiap variasi *payload stagless* untuk setiap *listener* pada direktori `C:\Payloads`.

![Generate Semua Payload](/assets/img/posts/cobalt-strike/11.png)
_Generate Semua Payload_

> Untuk mengetahui penjelasan lengkap mengenai *payload staged* dan *stageless*, saya merekomendasikan Anda untuk membaca artikel "[Staged vs Stageless Handlers](https://buffered.io/posts/staged-vs-stageless-handlers/)".
{: .prompt-info }

## 0x6 - Interacting with Beacon

Berikut adalah cara kita berinteraksi dengan *beacon*:

### A. Run Egress Beacon

Pertama-tama kita jalankan *payload* `http_x64.exe` pada mesin Windows lokal.

Terlihat pada gambar berikut, koneksi *beacon* dari mesin Windows lokal masuk ke Cobalt Strike.

![Egress Beacon](/assets/img/posts/cobalt-strike/12.png)
_Egress Beacon_

Keterangan:
- Kolom **sleep**: menunjukkan seberapa sering *beacon* harus melakukan *check-in* dengan Team Server. Secara default, *check-in* dilakukan setiap 60 detik. Penyesuaian *interval sleep* dapat membantu menghindari deteksi dan membuat komunikasi antara *beacon* dan Team Server lebih mirip dengan *traffic* jaringan normal.
- Kolom **last**: menunjukkan berapa lama waktu sejak *beacon* terakhir kali melakukan *check-in*. Ini dapat membantu kita untuk menentukan apakah *beacon* masih aktif atau tidak.
- Jika *beacon* melewatkan tiga *check-in* berturut-turut, angka di kolom terakhir akan menjadi tebal/miring untuk menunjukkan bahwa *beacon* mungkin telah hilang. Ini dapat berarti bahwa *beacon* telah ditutup, atau bahwa komunikasi antara *beacon* dan Team Server telah diblokir.

### B. Execute Command

Untuk menjalankan perintah pada *beacon*, klik kanan pada bagian koneksi *beacon*, lalu pilih **Interact**. Akan muncul tab baru untuk menjalankan perintah. 

![Tempat Menjalankan Perintah](/assets/img/posts/cobalt-strike/13.png)
_Tempat Menjalankan Perintah_

```bash
beacon> help

Beacon Commands
===============

    Command                   Description
    -------                   -----------
    !                         Run a command from the history
    argue                     Spoof arguments for matching processes
    blockdlls                 Block non-Microsoft DLLs in child processes
    browserpivot              Setup a browser pivot session
    cancel                    Cancel a download that's in-progress
    cd                        Change directory
    ...
```

Karena sedang mempelajari Cobalt Strike dan agar tidak menunggu lama, kita dapat memberi tahu *beacon* untuk melakukan *check-in* lebih sering dengan menggunakan perintah `sleep`.

```bash
beacon> sleep 5
beacon> sleep 0 # interactive
```

> Untuk mempermudah menggunakan Cobalt Strike kita bisa mempelajari [*keyboard shortcuts*](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/appendix-a_main.htm).
{: .prompt-info }

### C. Run Peer-To-Peer Beacon

Untuk mencoba P2P *beacon*, pertama jalankan `tcp-local_x64.exe`. 

Terlihat pada gambar berikut, tidak ada koneksi *beacon* baru yang masuk ke Cobalt Strike. Namun, pada Windows lokal yang menjalankan *beacon* terdapat port *beacon* yang terbuka sebagai *listener*.

![Peer-To-Peer Beacon](/assets/img/posts/cobalt-strike/14.png)
_Menjalankan Peer-To-Peer Beacon_

Untuk menghubungkan ke P2P *beacon* (*child*), jalankan perintah berikut pada *egress beacon* (*parent*):

```bash
beacon> connect <target> <port>
beacon> connect localhost 4444
```

![Menghubungkan Ke Peer-To-Peer Beacon](/assets/img/posts/cobalt-strike/15.png)
_Menghubungkan Ke Peer-To-Peer Beacon_

Jika kita menjalankan perintah `sleep` pada *child* atau *parent*, maka keduanya akan menerima perintah yang sama.

![Sleep Pada Child Beacon](/assets/img/posts/cobalt-strike/17.png)
_Sleep Pada Child Beacon_

### D. Graph View

Untuk melihat host yang terhubung secara visual, Klik **icon Graph**.

![Visualisasi Beacon Yang Terhubung](/assets/img/posts/cobalt-strike/16.png)
_Visualisasi Beacon Yang Terhubung_

Keterangan:
- *Green arrow* = TCP
- *Yellow arrow* = SMB


### E. Remove Beacon 

Untuk memutuskan koneksi *beacon*, jalankan perintah berikut:

```bash
beacon> exit
```

![Memutuskan Koneksi Beacon_](/assets/img/posts/cobalt-strike/18.png)
_Memutuskan oneksi Beacon_

Untuk menghilangkan *beacon* yang telah terputus, klik kanan pada koneksi *beacon*, lalu pilih **Session > Remove**.

![Menghapus Beacon](/assets/img/posts/cobalt-strike/19.png)
_Menghapus Beacon_

> Jika kita menghapus koneksi *egress beacon* yang sedang terhubung (*alive*), secara otomatis *beacon* akan mencoba menghubungkan kembali ke Team Server. Jadi kita tidak perlu khawatir jika tidak sengaja menghapus koneksi *beacon*.
{: .prompt-info }

## 0x7 - Pivot Listeners

*Pivot listener* memungkinkan kita untuk mengarahkan (*pivot*) *traffic* dari satu *beacon* ke *beacon* lain.

*Pivot listener* hanya dapat dibuat pada *beacon* yang sudah ada, dan tidak melalui menu **Listeners**.

*Pivot listener* bekerja dengan cara yang sama seperti *listener* TCP reguler, tetapi dalam urutan yang terbalik. 

*Beacon* TCP standar akan mengikat (*bind*) ke `127.0.0.1` (atau `0.0.0.0`) dan mendengarkan koneksi masuk pada port yang ditentukan. Kita kemudian memulai koneksi ke *beacon* tersebut dari *beacon* yang sudah ada (dengan perintah `connect`). 

Sementara itu, *pivot listener* bekerja sebaliknya dengan memberi tahu *beacon* yang sudah ada untuk mengikat (*bind*) dan mendengarkan pada port yang ditentukan, dan *beacon* TCP baru menginisiasi koneksi ke *beacon* tersebut.

Untuk membuat *pivot listener*, klik kanan pada koneksi *beacon* dan pilih **Pivoting > Listener**.

![Pivot Listener](/assets/img/posts/cobalt-strike/20.png)
_Pivot Listener_

Jenis *payload* yang digunakan untuk *pivot listener* adalah `beacon_reverse_tcp`, bukan `beacon_bind_tcp`. Meskipun ada menu *drop-down*, `beacon_reverse_tcp` adalah satu-satunya jenis *payload* yang saat ini dapat digunakan dengan *pivot listener*.

Opsi _listen host_ dan _listen port_ adalah detail koneksi yang akan disertakan dalam *payload* yang dihasilkan dari *listener* ini. Opsi ini secara otomatis diisi dari *beacon* yang kita pilih untuk bertindak sebagai *pivot*.

Setelah mengklik **save** dan menjalankan perintah `netstat`, kita akan melihat bahwa terdapat port *pivot listener* terbuka (dalam contoh ini, `1234`).

> Kita akan melihat peringatan firewall Windows. Dalam konteks tutorial ini, kita diminta untuk mengklik "*Allow Access*" untuk saat ini, dan taktik untuk menangani peringatan semacam ini akan dibahas pada materi selanjutnya.
{: .prompt-warning }

![Allow Access Firewall](/assets/img/posts/cobalt-strike/21.png)
_Allow Access Firewall_

![Port Pivot Listener Terbuka](/assets/img/posts/cobalt-strike/22.png)
_Port Pivot Listener Terbuka_

![Tampilan Pivot Listener](/assets/img/posts/cobalt-strike/23.png)
_Tampilan Pivot Listener_

Setelah *pivot listener* berhasil dibuat, selanjutnya kita buat *beacon* khusus untuk terhubung ke *pivot listener*.

![Generate Payload Pivot (1)](/assets/img/posts/cobalt-strike/25.png)
_Generate Payload Pivot (1)_

![Generate Payload Pivot (2)](/assets/img/posts/cobalt-strike/24.png)
_Generate Payload Pivot (2)_

Ketika *payload* dieksekusi, *reverse TCP beacon* akan muncul di Cobalt Strike. Di tampilan visual, panah menunjukkan arah koneksi.

![Contoh Pivot Listener](/assets/img/posts/cobalt-strike/26.png)
_Contoh Pivot Listener_

Untuk menghentikan *pivot listener*, buka menu **Listeners** reguler, sorot dan klik tombol **remove**.

Jika kita mengakhiri sesi *beacon* yang menjalankan *pivot listener*, maka *pivot listener* tidak akan bisa digunakan lagi.

![Pivot Listener Tidak Bisa Digunakan](/assets/img/posts/cobalt-strike/27.png)
_Pivot Listener Tidak Bisa Digunakan_

## 0x8 - Running as a Service

Agar menghemat waktu, kita bisa membuat Team Server berjalan sebagai *service* di server Linux. 

Dengan pendekatan ini kita tidak perlu khawatir jika komputer yang menjalankan Team Server ter-*restart*, karena secara otomatis *service* akan berjalan kembali. 

Untuk melakukan ini, pertama kita buat file di `/etc/systemd/system`.

```bash
➜ sudo vim /etc/systemd/system/teamserver.service
```

Lalu tambahkan *script* berikut:

```
[Unit]
Description=Cobalt Strike Team Server
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
WorkingDirectory=/home/attacker/cobaltstrike
ExecStart=/home/attacker/cobaltstrike/teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile

[Install]
WantedBy=multi-user.target
```

Selanjutnya, *reload* **systemd manager** dan periksa status *service*. Akan terlihat bahwa *service* itu *inactive/dead*.

```bash
➜ sudo systemctl daemon-reload
➜ sudo systemctl status teamserver.service
● teamserver.service - Cobalt Strike Team Server
     Loaded: loaded (/etc/systemd/system/teamserver.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
```

Jalankan *service* dan periksa kembali statusnya.

```bash
➜ sudo systemctl start teamserver.service
➜ sudo systemctl status teamserver.service
● teamserver.service - Cobalt Strike Team Server
     Loaded: loaded (/etc/systemd/system/teamserver.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-09-05 08:25:26 UTC; 14s ago
   Main PID: 1406 (teamserver)
      Tasks: 19 (limit: 4620)
     Memory: 47.5M
     CGroup: /system.slice/teamserver.service
             ├─1406 /bin/bash /home/attacker/cobaltstrike/teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile
             └─1447 ./TeamServerImage -Dcobaltstrike.server_port=50050 -Dcobaltstrike.server_bindto=0.0.0.0 -Djavax.net.ssl.keyStore=./cobaltstrike.store -Djavax.net.ssl.keyStorePassword=123456 teamserver >
Aug 05 08:25:28 ubuntu teamserver[1447]: [*] Setting 'https.protocols' system property: SSLv3,SSLv2Hello,TLSv1,TLSv1.1,TLSv1.2,TLSv1.3
Aug 05 08:25:28 ubuntu teamserver[1447]: [+] I see you're into threat replication. c2-profiles/normal/webbug.profile loaded.
Aug 05 08:25:37 ubuntu teamserver[1447]: [*] Loading Windows error codes.
Aug 05 08:25:37 ubuntu teamserver[1447]: [*] Windows error codes loaded
Aug 05 08:25:37 ubuntu teamserver[1447]: [*] Loading beacons
Aug 05 08:25:37 ubuntu teamserver[1447]: [*] Loaded 0 beacons
Aug 05 08:25:37 ubuntu teamserver[1447]: [+] Team server is up on 0.0.0.0:50050
Aug 05 08:25:37 ubuntu teamserver[1447]: [*] SHA256 hash of SSL cert is: 3bf25b6317a1c948cfad31faa0e14414d2d35f73b7947fa0bd3717ab5d0bc32d
Aug 05 08:25:37 ubuntu teamserver[1447]: [+] Listener: dns started!
Aug 05 08:25:37 ubuntu teamserver[1447]: [+] Listener: http started!
```

Sekarang *service* tersebut seharusnya *active/running*.

```bash
➜ sudo systemctl enable teamserver.service
Created symlink /etc/systemd/system/multi-user.target.wants/teamserver.service → /etc/systemd/system/teamserver.service.
```
