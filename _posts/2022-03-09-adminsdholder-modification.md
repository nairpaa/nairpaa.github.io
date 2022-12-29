---
title: AdminSDHolder Modification
date: 2022-03-09 23:26:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, adminsdholder, persistence]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

**AdminSDHolder Modification** adalah teknik persitensi di mana penyerang menyalahgunakan proses **SDProp** di Active Directory untuk membuat *backdoor* persisten ke Active Directory. 

Secara default, setiap jam SDProp membandingkan *permissions* pada object yang dilindungi (contoh: grup Domain Admins) di Active Directory dengan yang ditentukan pada *container* khusus yang disebut **AdminSDHolder**. Jika berbeda, **SDProp** akan menggantikan *permissions* pada object yang dilindungi dengan yang ditentukan oleh **AdminSDHolder**.

Dengan mengubah **AdminSDHolder**, penyerang dapat membuat jalur administrasi bayangan dan sarana untuk mendapatkan akses administratif kembali ke Active Directory.

List object user dan grup yang dilindungi:
- Account Operators
- Administrators
- Backup Operators
- Domain Admins
- Domain Controllers
- Administrator
- Enterprise Admins
- Print Operator
- Read-Only Domain Controllers
- Schema Admins
- Server Operator
- KRBTGT


### Tujuan 

- *Persistence*


### Prasyarat

- Penyerang memiliki akses ke DC (administratif)


### Tools

- *None*


## Tahap eksploitasi

### Step 1: Ubah Konfigurasi AdminSDHolder 

Tambahkan user yang digunakan penyerang di konfigurasi *security* **AdminSDHolder** dan berikan hak akses 'Full control'.

![Mengubah konfigurasi AdminSDHolder](/assets/img/posts/adminsdholder-modification/1.png)


### Step 2: Tunggu proses SDProp 

Setelah itu tunggu proses SDProp (default-nya 60 menit) dan konfigurasi object user dan grup yang dilindungi akan terubah.

![Hasil menunggu SDProp](/assets/img/posts/adminsdholder-modification/2.png)


## Mitigasi

AdminSDHolder container adalah bagian inti dari Active Directory. Secara default, hanya pengguna administratif yang dapat mengubah konfigurasi ACL-nya. 

Untuk mengurangi risiko modifikasi yang tidak sah, ada beberapa cara yang dapat dilakukan, seperti:
- Audit secara rutin *permissions* dari AdminSDHolder  
- Lakukan pencegahan serangan Active Directory, seperti tidak menggunakan Domain Admins pada komputer user.

--- 

## Referensi

- https://attack.stealthbits.com/adminsdholder-modification-ad-persistence
- https://specopssoft.com/support/password-reset/understanding-privileged-accounts-and-adminsdholder.htm
- https://adsecurity.org/?p=1906