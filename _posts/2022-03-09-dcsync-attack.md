---
title: DCSync Attack
date: 2022-03-09 00:20:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, dcsync attack]     # TAG names should always be lowercase
author: nairpaa
---

**DCSync** adalah teknik *dumping* kredensial yang dapat mengarah pada kompromisasi kredensial pengguna, dan lebih serius sebagai pendahuluan untuk pembuatan **Golden Ticket**, karena **DCSync** dapat digunakan untuk mengkompromisasikan kata sandi akun **krbtgt**.

Skenario penyerangan ini adalah penyerang mengkompromisasikan akun yang memiliki hak akses untuk melakukan replikasi domain. 

Selanjutnya dengan menggunakan perintah **Mimikatz DCSync**, penyerang mendapatkan hash dari akun Active Directory (**krbtgt**). 

Setelah diperoleh, penyerang dapat membuat ticket Kerberos palsu untuk mengakses *resource* apa pun yang terhubung ke Active Directory.

### Tujuan 

- *Privilege Escalation*


### Prasyarat

- Untuk melakukan serangan ini penyerang harus mendapatkan akun yang memiliki hak akses `Replicating Directory Changes All` dan `Replicating Directory Changes`.

![Konfigurasi Replication Directory Changes](/assets/img/posts/dcsync-attack/1.png)

### Tools

- Mimikatz
- Impacket

## Tahap eksploitasi

### Via remote (Kali)

```bash
➜ secretsdump.py <domain>/<username>:<password>@<ip-dc>
```

### Via internal (Windows)
Jalankan perintah berikut untuk mereplikasi kredensial dari Active Directory. Target paling umum untuk replikasi adalah akun **krbtgt**, karena password akun ini merupakan prasyarat untuk **Golden Ticket**.
```bash
mimikatz > lsadump::dcsync /domain:<domain> /user:krbtgt
```

![DCSync pada Mimikatz](/assets/img/posts/dcsync-attack/2.png)


## Migitagasi

Hati-hati dengan penggunaan hak akses pada `Replicating Directory Changes All` dan `Replicating Directory Changes`. Pastikan yang memiliki hak akses tersebut adalah pengguna yang terpercaya. 

---

## Referensi

- https://adsecurity.org/?p=1729
- https://attack.stealthbits.com/privilege-escalation-using-mimikatz-dcsync
- https://www.qomplx.com/kerberos_dcsync_attacks_explained/
- https://www.youtube.com/watch?v=aSAZzIqGeiY
- https://yojimbosecurity.ninja/dcsync/#org4679e78
- https://burmat.gitbook.io/security/hacking/domain-exploitation
