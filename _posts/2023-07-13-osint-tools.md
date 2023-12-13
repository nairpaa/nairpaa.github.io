---
title: '02 - OSINT Tools'
date: 2023-07-13 17:09:22 +0700
categories: ['Red Team', 'External Reconnaissance']
tags: [osint, collection tools]     # TAG names should always be lowercase
author: nairpaa
---

Pada bagian ini saya mengumpulkan *tools* yang digunakan untuk melakukan aktivitas OSINT.

## 0x1 - Note Keeping

Kumpulan aplikasi yang membantu untuk mencatat informasi yang dikumpulkan selama aktivitas OSINT.

**Note Apps**
- [Obsidian](https://obsidian.md/)
- [KeepNote](https://keepnote.org/)
- [Cherrytree](https://www.giuspen.com/cherrytree/)
- [Joplin](https://joplinapp.org/)
- [Notion](https://www.notion.so/)
- [OneNote](https://onenote.com/)

**Screenshoot App**
- [Greenshot](https://getgreenshot.org/)
- [Flameshot](https://github.com/flameshot-org/flameshot)

**Bookmark App**
- [Start.me](https://start.me/)

**Mind Map App**
- [Xmind](https://xmind.app/)

## 0x2 - Sock Puppets

**Sock Puppets** dalam konteks OSINT mengacu pada identitas online palsu atau anonim yang dibuat dan digunakan untuk mengumpulkan informasi atau berinteraksi dengan orang lain tanpa mengungkapkan identitas sebenarnya.

Misalnya, seorang profesional OSINT mungkin membuat *sock puppet* untuk bergabung dengan forum online atau grup media sosial yang terkait dengan subjek investigasi mereka. *Sock puppet* ini memungkinkan mereka untuk memantau diskusi, mengajukan pertanyaan, atau mungkin bahkan mengumpulkan informasi langsung dari individu atau organisasi lainnya.

Berikut adalah beberapa artikel dan *tools* yang bisa kita gunakan untuk membuat *sock puppet*:

- [Creating an Effective Sock Puppet for OSINT Investigations](https://web.archive.org/web/20210125191016/https://jakecreps.com/2018/11/02/sock-puppets)
- [The Art Of The Sock](https://www.secjuice.com/the-art-of-the-sock-osint-humint/)
- [My process for setting up anonymous sockpuppet accounts](https://www.reddit.com/r/OSINT/comments/dp70jr/my_process_for_setting_up_anonymous_sockpuppet/)
- [Fake Name Generator](https://www.fakenamegenerator.com)
- [Fake Name Generator (Indonesia)](https://www.random-name-generator.com/indonesia)
- [Fake Name Generator (fauxid)](https://fauxid.com/fake-name-generator/indonesia)
- [This Person Does not Exist](https://www.thispersondoesnotexist.com/)
- [This Person Does not Exist (Asia)](https://this-person-does-not-exist.com/)
- [Privacy.com](https://privacy.com/)

## 0x3 - DNS Records

**Domain and IP Information**

```bash
➜ dig example.com # mencari alamat IP dari domain
➜ whois 93.184.216.34 # mengetahui informasi dari IP dan domain ketika didaftarkan
```

- [whois](https://www.tecmint.com/whois-command-get-domain-and-ip-address-information/)
- [dig](https://www.geeksforgeeks.org/dig-command-in-linux-with-examples/)
- [DNS Dumpster](https://dnsdumpster.com/)
- [IP Info](https://ipinfo.io/)
- [VirusTotal](https://www.virustotal.com/)
- [Domain Dossier](https://centralops.net/co/)
- [DNSlytics](https://dnslytics.com/reverse-ip)
- [View DNS](https://viewdns.info/)
- [Robtex](https://www.robtex.com/)

**Sub-Domain Enumeration**

```bash
➜ ./dnscan.py -d example.com -w subdomains-100.txt
➜ aquatone-discover -d example.com
➜ amass enum -passive -d example.com
➜ subfinder -d example.com
➜ sudo docker run -v "${PWD}/output:/usr/lib/sudomy/output" -v "/home/kali/.config/sudomy/sudomy.api:/usr/lib/sudomy/sudomy.api" -t --rm screetsec/sudomy:v1.2.0-dev -d example.com
➜ assetfinder example.com
```

- [dnscan](https://github.com/rbsec/dnscan)
- [amass](https://github.com/owasp-amass/amass)
- [aquaton](https://www.geeksforgeeks.org/aquatone-tool-for-domain-flyovers/)
- [assetfinder](https://github.com/tomnomnom/assetfinder)
- [subfinder](https://github.com/projectdiscovery/subfinder)
- [sudomy](https://github.com/screetsec/Sudomy)
- [crt.sh](https://crt.sh/)
- [Netcraft](https://searchdns.netcraft.com/)
- [Facebook](https://developers.facebook.com/tools/ct/)

**Spoofing Check**

```bash
➜ python3 spoofcheck.py example.com # check mail spoofing
```

- [spoofcheck](https://github.com/a6avind/spoofcheck)
- [Emkei's Fake Mailer](https://emkei.cz/)

## 0x4 - Google Dorks

- [Google Hacking Database](https://www.exploit-db.com/google-hacking-database)
- [Google Hacking by Pentest Tools](https://pentest-tools.com/information-gathering/google-hacking)

## 0x5 - Social Media OSINT

**LinkedIn**
- [LinkedInt](https://github.com/mdsecactivebreach/LinkedInt)

**Facebook**
- [Sowdust Github](https://sowdust.github.io/fb-search/)
- [IntelligenceX Facebook Search](https://intelx.io/tools?tab=facebook)

**Instagram**
- [Osintgram](https://github.com/Datalux/Osintgram)
- [Wopita](https://wopita.com/)
- [Code of a Ninja](https://codeofaninja.com/tools/find-instagram-user-id/)
- [InstaDP](https://www.instadp.com/)
- [ImgInn](https://imgsed.com/)

**Twitter**
- [Twitter Advanced Search](https://twitter.com/search-advanced)
- [Social Bearing](https://socialbearing.com/)
- [Twitonomy](https://www.twitonomy.com/)
- [Mentionmapp](https://mentionmapp.com/)
- [Tweetbeaver](https://tweetbeaver.com/)
- [Tinfoleak](https://tinfoleak.com/)
- [TweetDeck](https://tweetdeck.com/)

**Snapchat**
- [Snapchat Maps](https://map.snapchat.com/)

**Another**
- [Profil3r](https://github.com/MrNonoss/Profil3r-docker)
- [sherlock](https://github.com/sherlock-project/sherlock)
- [Instagram & Twitter OSINT - DownUnderCTF by John Hammond](https://www.youtube.com/watch?v=DV8hUcdK2Bk)
- [Investigasi Seseorang Lewat Jejak Digital with Sherlock](https://www.youtube.com/watch?v=7qGIg9Oj9ko)
- [Instagram Osint Using Osintgram](https://www.youtube.com/watch?v=z7xW0VfxNNI)
- [Social Media Investigation with Profil3r](https://www.youtube.com/watch?v=OsBQ6p3rpDo)

## 0x6 -  Image OSINT

**Reverse Image Searching**
- [Google Image Search](https://images.google.com)
- [Yandex](https://yandex.com)
- [TinEye](https://tineye.com)

**View EXIF Data**

```bash
➜ exiftool example.png
```
- [exiftool](https://exiftool.org/)

**Identifying Geopgraphical Locations**
- [GeoGuessr](https://www.geoguessr.com)
- [GeoGuessr - The Top Tips, Tricks and Techniques](https://somerandomstuff1.wordpress.com/2019/02/08/geoguessr-the-top-tips-tricks-and-techniques/)

## 0x7 - Email OSINT

- [Hunter.io](https://hunter.io/)
- [Phonebook.cz](https://phonebook.cz/)
- [VoilaNorbert](https://www.voilanorbert.com/)
- [Email Hippo](https://tools.verifyemailaddress.io/)
- [Email Checker](https://email-checker.net/validate)
- [Clearbit Connect](https://chrome.google.com/webstore/detail/clearbit-connect-supercha/pmnhcgfcafcnkbengdcanjablaabjplo?hl=en)

## 0x8 - Password OSINT
- [Dehashed](https://dehashed.com/)
- [WeLeakInfo](https://weleakinfo.to/v2/) 
- [LeakCheck](https://leakcheck.io/)
- [SnusBase](https://snusbase.com/)
- [Scylla.sh](https://scylla.sh/)
- [HaveIBeenPwned](https://haveibeenpwned.com/)

## 0x9 - Username OSINT

- [NameChk](https://namechk.com/)
- [WhatsMyName](https://whatsmyname.app/)
- [NameCheckup](https://namecheckup.com/)

## 0x10 - People OSINT

**Searching for People**
- [WhitePages](https://www.whitepages.com/)
- [TruePeopleSearch](https://www.truepeoplesearch.com/)
- [FastPeopleSearch](https://www.fastpeoplesearch.com/)
- [FastBackgroundCheck](https://www.fastbackgroundcheck.com/)
-  [WebMii](https://webmii.com/)
- [PeekYou](https://peekyou.com/)
- [411](https://www.411.com/)
- [Spokeo](https://www.spokeo.com/)
- [That'sThem](https://thatsthem.com/)

**Voter Records**
- [Voter Records](https://www.voterrecords.com)

**Hunting Phone Numbers**
- [TrueCaller](https://www.truecaller.com/)
- [CallerID Test](https://calleridtest.com/)
- [Infobel](https://infobel.com/)

## 0x11 - Website OSINT

 - [Visual Ping](https://visualping.io/)
 - [BuiltWith](https://builtwith.com/)
 - [SpyOnWeb](https://spyonweb.com/)
 - [Back Link Watch](https://backlinkwatch.com/index.php)
 - [Wayback Machine](https://web.archive.org/)

## 0x12 - Business OSINT

- [Open Corporates](https://opencorporates.com/)
- [AI HIT](https://www.aihitdata.com/)

## 0x13 - Wireless OSINT

- [WiGLE](https://wigle.net/)

## 0x14 - Network OSINT

- [nrich Shodan](https://gitlab.com/shodan-public/nrich)
- [Grabify IP Logger](https://grabify.link/3ZOCMI)