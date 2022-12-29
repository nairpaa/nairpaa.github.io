---
title: Part 4 - LLMNR Poisoning
date: 2022-02-05 09:58:22 +0800
categories: [Active Directory, Initial Attack Vector]
tags: [active directory, windows, responder, hashcat, poisoning]     # TAG names should always be lowercase
author: nairpaa
---

### Tujuan 

- *Get a foothold user*

### Prasyarat

- Menunggu kesalahan user


### Tools

- Responder
- Hashcat

---

## Apa itu LLMNR?

**Name Resolution** (singkatan **NR**) adalah serangkaian prosedur yang dilakukan oleh komputer untuk mengambil alamat IP host dengan nama host-nya. Pada Windows, prosedurnya seperti berikut:
- Alamat IP hostname *file share* diperlukan.
- Komputer akan mencari ke **hostfile** lokal  untuk mengecek *record name server*.
- Jika pada **hostfile** tidak ditemukan, komputer akan mencari ke *cache* lokal DNS untuk mengecek *record name server*.
- Tidak ada di *cache* lokal DNS? Komputer akan mencari *record name server* ke DNS server.
- Jika masih tidak ada, komputer akan mengirimkan kueri *multicast*, menanyakan alamat IP *file share* ke komputer lain dalam jaringan.

Seperti yang bisa kita lihat, *fallback* terakhir saat melakukan *resolving name* alamat IP adalah menggunakan NR *multicast*. NR ini dikelola oleh tiga protokol, yaitu: **NBT-NS** (NetBIOS Name Service), **LLMNR** (Link-Local Multicast Name Resolution) dan **mDNS** (multicast DNS).


Ketiga protokol tersebut digunakan secara tambahan karena dua alasan utama: dukungan dan kompatibilitas lama.
- **NBT-NS** dibuat pada tahun 80-an dan agak tidak sesuai dengan standar saat ini.
- Sebagai penggantinya, terdapat **LLMNR** (yang masih mendukung **NBT-NS** untuk berkomunikasi dengan komputer lama). 
- Di sisi lain, kebanyakan komputer berbasis Linux mengimplementasikan **mDNS** sebagai gantinya. Akhirnya dengan merilis Windows 10, Microsoft menambahkan dukungan **mDNS** untuk meningkatkan kompatibilitas secara keseluruhan.

![Cara kerja llmnr poisoning](/assets/img/posts/llmnr-poisoning/1.png)


## Kenapa ini menjadi masalah?

**NBT-NS**, **LLMNR** dan **mDNS** melakukan *broadcast* kueri ke seluruh jaringan, tetapi tidak ada tindakan yang diambil untuk memverifikasi integritas respons. 

Penyerang dapat mengeksploitasi kerentanan ini dengan mendengarkan pertanyaan dan melakukan *spoofing response* (menipu korban agar mempercayai server penyerang). 

Biasanya serangan ini digunakan untuk mencuri NTLMv2 hash.

### Contoh Umum Penyalahgunaan

- **Mistyping** - Jika user salah mengetik nama host, biasanya tidak ada *record* host yang relevan dan komputer akan menggunakan NR multicast.
- **Misconfiguring** - Konfigurasi yang salah pada server DNS atau sisi klien dapat menyebabkan masalah, sehingga memaksa klien untuk menggunakan NR multicast.
- **WPAD Protocol** - Jika browser diatur secara otomatis mendeteksi pengaturan proxy, komputer akan menggunakan protokol WPAD dan melakukan iterasi melalui serangkaian URL dan *hostname* potensial. (Google Chrome dan Firefox tidak akan memicu perilaku ini secara default, tetapi Internet Explorer akan melakukannya.)
- **Google Chrome** - Ketika satu string kata diketik di *search bar* chrome, aplikasi membutuhkan cara untuk membedakan apakah string tersebut adalah URL atau istilah pencarian. Chrome pertama-tama memperlakukan string sebagai istilah pencarian dan mengarahkan pengguna ke *search engine*, sekaligus memastikan string tersebut bukan nama host dengan mencoba me-*resolve*-nya. Selanjutnya, untuk mencegah paparan terhadap DNS *hijacking*, Chrome akan mencoba me-*resolve* beberapa hostname acak saat memulai untuk memastikan mereka tidak dapat me-*resolve*-nya – pada dasarnya ini menjamin beberapa tindakan NR multicast.

## Tahap eksploitasi

### Step 1: Pastikan kita satu jaringan dengan korban

![Contoh topologi untuk melakukan llmnr poisoning](/assets/img/posts/llmnr-poisoning/2.png)


### Step 1: Jalankan responder
Penyerang menjalankan resonder:
```bash
➜ sudo responder -I eth0 -rdw 
```

> Konfigurasi responder untuk layanan HTTP dan SMB diaktifkan. 
{: .prompt-note }

### Step 2: Tunggu korban melakukan kesalahan

Tunggu korban melakukan kesalahan penulisan *hostname*.
Sebagai contoh, korban melakukan salah ketik, yaitu mengakses `\\tidakada\` .

![Contoh korban melakukan kesalahan](/assets/img/posts/llmnr-poisoning/3.png)

### Step 3: Dapatkan NTLMv2 Hash

Karena `tidakada` tidak tersedia di *hostfile*, lokal *cache* DNS dan DNS server, komputer akan melakukan NR *multicast* ke semua komputer dalam jaringan. 

Ketika korban mengirimkan kueri kepada komputer penyerang, **responder** akan melakukan penipuan terhadap korban, seolah-olah benar ia adalah target yang si korban tuju.

Setelah itu, korban akan memberikan NTLMv2 hash-nya kepada penyerang.

![Mendapatkan NTLMv2 hash dari llmnr poisoning](/assets/img/posts/llmnr-poisoning/4.png)

### Step 4: Crack de hash

NTLMv2 dapat di-*crack* menggunakan `hashcat` dengan perintah berikut:

```bash
➜ hashcat -m 5600 pass.hash /usr/share/wordlists/rockyou.txt 
```


## Mitigasi

Karena NR multicast adalah perilaku *peer-to-peer*, sebagian besar metode mitigasi akan berfokus pada keamanan *endpoint*, daripada hanya mengandalkan keamanan jaringan:

- **Disabling LLMNR** - LLMNR dapat dimatikan melalui *group policy editor*, pada menu **Local Computer Policy** > **Computer Configuration** > **Administrative Templates** > **Network** > **DNS Client**.
- **Disabling NBT-NS** - NBT-NS dapat dimatikan melalui **Network Connection Settings**. Arahkan ke **Network Connections** > **Internet Protocol Version 4** > **Properties** > **General** > **Advanced** > **WINS**, lalu pilih "**Disable NetBIOS over TCP/IP**". 
- **Network Traffic Filtration** - produk keamanan host-based dapat membantu untuk memblokir *traffic* LLMNR, NBT-NS dan mDNS.
- **SMB Signing** - dapat membantu mencegah serangan NTLM *relay* dengan menandatangani data yang ditransfer secara digital.
- **Monitoring** - host harus dipantau (1) untuk *traffic* pada port LLMNR dan NBT-NS (UDP 5355 dan 137), (2) *event logs* dengan *event ID* 4697 and 7045 (*relevant to relay attacks*) dan (3) perubahan registry `DWORD EnableMulticast` di `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient`.
- **Strong password** - Penggunaan kata sandi yang kuat ( > 14 karakter dan hindari penggunaan kata yang umum). Semakin kompleks dan panjang kata sandi, semakin sulit bagi penyerang untuk memecahkan hash.


## Referensi

- [How to Disable LLMNR, Netbios, WPAD, & LM Hash](https://www.blumira.com/integration/disable-llmnr-netbios-wpad-lm-hash/)
- [LLMNR & NBT-NS Poisoning and Credential Access using Responder](https://www.cynet.com/attack-techniques-hands-on/llmnr-nbt-ns-poisoning-and-credential-access-using-responder/)