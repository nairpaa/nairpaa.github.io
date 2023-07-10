---
title: '0x2 - What is Red Teaming?'
date: 2023-07-08 14:00:22 +0700
categories: ['Red Team', 'Fundamental']
tags: [red team, fundamental, what is]     # TAG names should always be lowercase
author: nairpaa
---

Pada bagian ini, kita akan mempelajari konsep dasar yang diperlukan untuk mendalami *Red Teaming* sebelum masuk ke bagian teknis.

## 0x1 - What is Red Teaming?

Istilah "**Red Team**" berasal dari terminologi militer, khususnya dalam latihan atau simulasi perang, di mana "*Red Force*" sering digunakan untuk mewakili musuh atau penyerang, sedangkan "*Blue Force*" mewakili pasukan yang bertahan.

Tujuan dibentuknya *Red Team* ini adalah untuk melatih dan mengukur keefektifan orang-orang, proses, dan teknologi (*Blue Team*) untuk menjaga keamanan.

Dalam konteks ini:
- **Orang-orang (*People*)**: merujuk kepada tim keamanan dan staf lainnya yang bertanggung jawab untuk menjaga keamanan.
- **Proses (*Process*)**: merujuk kepada prosedur dan kebijakan keamanan yang telah diterapkan oleh organisasi.
- **Teknologi (*Technology*)** merujuk kepada *tools* dan solusi keamanan yang digunakan.

Konsep *Red Teaming* ini telah diterapkan dalam berbagai konteks di luar militer, termasuk *Cyber Security*. 

Di dunia *Cyber Security*, *Red Team* adalah kelompok yang bertindak sebagai penyerang dalam skenario simulasi, dengan tujuan untuk melatih dan mengukur keefektifan *Blue Team* untuk melindungi sistem.

*Red Team* akan menggunakan Taktik, Teknik, dan Prosedur (TTPs) untuk meniru ancaman dunia nyata. Hal ini melibatkan penggunaan *tools*, metode, dan strategi yang serupa dengan penyerang yang sebenarnya.

Tujuannya adalah untuk membuat skenario pengujian yang realistis dan relevan dengan ancaman nyata yang dihadapi organisasi.

## 0x2 - What is TTPs?

TTPs (*Tactics, Techniques, and Procedures*) adalah istilah yang digunakan dalam komunitas keamanan siber dan militer untuk merujuk kepada metode yang digunakan oleh penyerang atau ancaman.

Berikut adalah penjelasan lebih detail tentang TTPs:
- **Taktik (*Tactics*)**: merujuk kepada strategi atau pendekatan secara keseluruhan yang digunakan oleh penyerang. Misalnya, penyerang mungkin memutuskan untuk menggunakan serangan *phishing* sebagai bagian dari kampanye mereka.
- **Teknik (*Techniques*)**: adalah bagaimana taktik tersebut dijalankan. Menggunakan contoh *phishing*, teknik mungkin melibatkan penyerang yang mengirim email yang mencoba meniru email dari bank atau perusahaan terpercaya.
- **Prosedur (*Procedures*)**: adalah langkah-langkah rinci yang digunakan untuk melaksanakan teknik. Menggunakan contoh yang sama, prosedur mungkin melibatkan pembuatan domain web palsu yang mirip dengan bank atau perusahaan yang ditiru, membuat email yang tampak meyakinkan, dan kemudian mengirim email ke target.

## 0x3 - How to learn about TTPs?

Terdapat berbagai sumber yang bisa digunakan oleh *Red Team* untuk mendapatkan informasi tentang TTPs yang sudah ada, diantaranya:
- **Laporan *threat intelligence***: Banyak organisasi dan perusahaan keamanan siber mempublikasikan laporan intelijen ancaman yang mendetail tentang TTPs yang digunakan oleh berbagai aktor ancaman. Laporan-laporan ini biasanya berdasarkan analisis insiden keamanan nyata dan penelitian keamanan.
- **Komunitas dan forum online**: Banyak komunitas dan forum online di mana peneliti keamanan dan profesional IT berbagi informasi tentang TTPs. Ini dapat mencakup forum resmi (seperti milis) atau platform sosial media (seperti Twitter atau Reddit).
- **Analisis *malware***: Dengan menganalisis kode dari *malware* yang ditemukan dalam insiden keamanan, peneliti dapat memahami bagaimana aktor ancaman beroperasi. Mereka bisa memahami cara kerja *malware*, bagaimana ia menghindari deteksi, dan bagaimana ia mencapai tujuannya.
- ***Framework* seperti MITRE ATT&CK**: [MITRE ATT&CK](https://attack.mitre.org/) adalah pengetahuan yang terus diperbarui dan mencakup taktik dan teknik yang digunakan oleh aktor ancaman dalam kampanye mereka. Hal ini bisa menjadi sebagai sumber daya yang sangat berharga bagi *Red Team*.

Sebelum melangkah lebih lanjut, kita harus mehamami bahwa *Red Team* tidak hanya meniru TTPs yang sudah ada dan tidak mencari yang baru.

Sebenarnya, bagian dari pekerjaan *Red Teaming* adalah melakukan penelitian dan pengembangan sendiri untuk mencari cara-cara baru dan inovatif untuk mengeksploitasi sistem dan jaringan.

Informasi ini kemudian dapat digunakan untuk meningkatkan pertahanan organisasi dan juga bisa ditambahkan ke basis pengetahuan yang digunakan oleh komunitas keamanan secara luas.

## 0x4 - Red Team Engagement vs Penetration Test

*Red Team Engagement* sering kali disamakan dengan *Penetration Testing*, tetapi keduanya tidak lah sama.

*Red Teaming* memiliki beberapa atribut yang membedakannya dengan *offensive security* yang lain, yang paling utama diantaranya:
- **Emulation of the TTPs**: yaitu melakukan emulasi serangan menggunakan metode yang sama seperti penyerang sebenarnya.
- **Campaign-based testing**: adalah pendekatan di mana serangan disimulasikan selama periode waktu yang cukup lama, biasanya beberapa minggu atau bulan. Ini berarti bahwa serangan tidak hanya terjadi sekali, tetapi sebagai bagian dari kampanye yang berkelanjutan yang meniru bagaimana penyerang sebenarnya mungkin beroperasi.

Kedua aspek penting tersebut membantu membuat simulasi serangan menjadi lebih realistis dan memberikan gambaran yang lebih baik tentang bagaimana organisasi akan merespons serangan nyata.

> Untuk lebih memahami perbedaan *Red Team Engagement* dengan tipe *Security Assessment* lainnya, Anda bisa membaca artikel-artikel berikut:
> - [Security Assessement Types](https://danielmiessler.com/p/security-assessment-types/)
> - [The Difference Between Red, Blue, and Purple Teams](https://danielmiessler.com/p/red-blue-purple-teams/)
> - [Five Attributes Effective Corporate Red Team](https://danielmiessler.com/p/five-attributes-effective-corporate-red-team/)
> - [Red Team Engagement vs Penetration Test vs Vulnerability Assessment](https://redteam.guide/docs/Concepts/red-vs-pen-vs-vuln)
{: .prompt-tip }


## 0x5 - Why Red Team?

Hasil dari *Red Teaming* adalah memang dapat mengidentifikasi kerentanan, tetapi yang lebih penting, *Red Teaming* memberikan pemahaman tentang sejauh mana kemampuan *Blue Team* dalam mendeteksi, mencegah, dan merespon ancaman.

![IT Security: Perception vs Reality](http://redteam.guide/assets/images/threat_gets_a_vote-6724612fc1058a2b4e0a729a0d1bd348.png){: w="500" h="400" }
_IT Security: Perception vs Reality_

*Red Team* memberikan perspektif yang berlawanan dengan menantang asumsi yang dibuat oleh organisasi atau *Blue Team*.

Asumsi yang dimaksud seperti:
- "Kita aman karena sudah melakukan *update* dan *patching*"
- "Hanya orang tertentu yang bisa mengakses sistem itu"
- "Kita sudah pakai teknologi X, pasti bisa menghentikan serangan"

Asumsi-asumsi tersebut adalah asumsi yang berbahaya dan sering kali tidak bisa dipertanggungjawabkan. 

Dengan menantang asumsi-asumsi tersebut, *Red Team* dapat mengidentifikasi area yang perlu ditingkatkan dalam pertahanan operasional organisasi.

Dari penjelasan tersebut kita jadi tahu kenapa *Red Team* diperlukan, yaitu diantaranya:
- **Mengukur efektivitas orang, proses, dan teknologi** yang digunakan untuk mempertahankan sistem. Bagaimana kita tahu jika *Blue Team* sudah efektif?
- **Melatih dan/atau mengukur kemampuan *Blue Team*** untuk menghadapi ancaman agar lebih siap menghadapi ancaman yang nyata.
- **Menguji dan memahami ancaman atau skenario tertentu**. Keterlibatan *Red Team* dapat dirancang untuk melatih skenario khusus. Skenario dapat mencakup *zero-days*, serangan *ransomware*, atau serangan unik lainnya.

## 0x6 - When to Use Red Team Services?

Layanan *Red Team* biasanya digunakan oleh organisasi yang sudah memiliki tingkat kematangan keamanan siber yang cukup tinggi, dengan sistem manajemen kerentanan yang kuat dan kemampuan untuk mendeteksi dan merespons perilaku mencurigakan atau jahat dalam lingkungan mereka.

Hal ini karena, jika organisasi masih berjuang dengan manajemen aset dasar, pembaruan *software* (*patching*), kontrol lalu lintas keluar (*egress traffic*), dan fundamental lainnya, biasanya lebih baik mereka memecahkan masalah ini terlebih dahulu sebelum menyewa atau membentuk "*Red Team*". 

*Red Team* dirancang untuk menguji postur keamanan yang matang dalam cara yang mendekati ancaman dunia nyata, bukan untuk mengidentifikasi masalah dalam lingkungan dengan tingkat kematangan rendah.

Dalam istilah sederhana, jika organisasi Anda tidak memiliki "*Blue Team*" (tim yang bertanggung jawab untuk mendeteksi dan merespons ancaman keamanan), maka kemungkinan besar Anda tidak memerlukan *Red Team*.

Perlu diingat bahwa tujuan dari *Red Teaming* adalah untuk melatih *Blue Team*.

Artinya, sebelum Anda berinvestasi dalam tes serangan simulasi yang canggih (seperti yang dilakukan oleh *Red Team*), Anda harus memastikan bahwa Anda sudah memiliki dasar-dasar keamanan siber yang kuat dan tim yang mampu merespons insiden keamanan.

Ini adalah pendekatan yang logis dan bertanggung jawab, karena jika dasar-dasar belum diperbaiki, hasil dari pengujian *Red Team* mungkin tidak memberikan manfaat maksimal bagi organisasi, dan sebaliknya mungkin hanya memperjelas apa yang sudah diketahui: bahwa dasar-dasar keamanan siber perlu diperbaiki.

## 0x7 - What is OPSEC?

**Operations Security (OPSEC)** adalah proses dan strategi yang digunakan untuk melindungi informasi yang bisa digunakan oleh penyerang untuk merencanakan dan melancarkan serangan. 

OPSEC mencakup identifikasi dan perlindungan informasi yang, jika jatuh ke tangan yang salah, bisa membahayakan operasi dan/atau keamanan suatu organisasi.

Untuk *Red Team*, OPSEC biasanya melibatkan upaya untuk menjaga kerahasiaan dan anonimitas selama melakukan simulasi serangan. Ini dapat mencakup taktik seperti menyembunyikan lokasi fisik mereka, menggunakan teknik pengecekan untuk menghindari deteksi, atau merancang serangan mereka sedemikian rupa sehingga sulit dilacak kembali. 

Tujuan utama dari OPSEC dalam konteks ini adalah untuk membuat serangan simulasi mereka seautentik mungkin, sehingga memberikan uji coba yang akurat dan efektif dari sistem keamanan organisasi.

Baik *Red Team* maupun *Blue Team* harus berasumsi bahwa tindakan mereka sedang dipantau atau diganggu oleh pihak lawan. Operator yang bijak juga akan berasumsi bahwa tim yang mereka hadapi lebih baik dari mereka.

## 0x8 - Primum non nocere

Jika Anda telah membaca artikel [Five Attributes Effective Corporate Red Team](https://danielmiessler.com/p/five-attributes-effective-corporate-red-team/), terdapat salah satu atribut penting pada *Red Team*, yaitu **Organizational Independence**.

Atribut ini menjelaskan bahwa *Red Team* harus memiliki kebebasan untuk bertindak seperti penyerang nyata dalam hal cakupan, *tools*, dan teknik yang digunakan. Namun, ini tidak berarti bahwa *Red Team* memiliki izin untuk melakukan apapun yang mereka inginkan tanpa batas.

Sebagai bagian dari pendekatan yang etis dan bertanggung jawab, setiap aktivitas *Red Teaming* harus didasarkan pada persetujuan dan aturan yang telah ditetapkan sebelumnya, biasanya dalam bentuk *rules of engagement* yang jelas. Aturan ini ditujukan untuk melindungi organisasi dan individu yang terlibat dari kerusakan atau gangguan yang tidak perlu.

Di dunia [kedokteran](https://en.wikipedia.org/wiki/Hippocratic_Oath) terdapat ungkapan latin "*primum non nocere*" yang berarti "*first, do no harm*". 

Idenya adalah bahwa dokter tidak boleh mengambil risiko melakukan hal yang lebih berbahaya daripada kebaikan bagi pasien mereka.

Prinsip ini sangat relevan bagi profesional keamanan siber. Tujuannya adalah untuk meningkatkan keamanan klien mereka, bukan untuk membahayakan atau melemahkan sistem keamanan mereka.

Dalam praktiknya hal ini tidak selalu sederhana. Mirip dengan dokter yang terkadang harus menyakiti pasien mereka untuk kebaikan jangka panjang (seperti operasi), anggota *Red Team* mungkin perlu melakukan tindakan yang berpotensi merugikan dalam rangka untuk menguji dan meningkatkan keamanan sistem.

Contohnya adalah menonaktifkan kontrol keamanan, menambahkan pengguna ke grup yang memiliki hak istimewa, atau membuat *backdoor* administratif ke sistem.

Hal ini dapat berbahaya karena kita tidak dapat menjamin bahwa tindakan tersebut tidak akan disalahgunakan oleh pihak lain, sehingga meningkatkan risiko bagi klien. Namun, taktik-taktik ini sering digunakan oleh penyerang asli, dan *Red Team* seharusnya berusaha meniru mereka.

Oleh karena itu, penilaian profesional dan bijaksana diperlukan. 

Sebagai contoh, *Red Team* tidak boleh benar-benar melakukan serangan *ransomware* ke seluruh organisasi hanya karena itu mungkin dilakukan oleh penyerang asli. Jika klien ingin menggunakan skenario tersebut, itu harus dilakukan dengan cara yang aman dan terkontrol.

Intinya, seperti seorang dokter yang tidak akan melakukan operasi tanpa persetujuan pasien, *Red Team* juga harus menahan diri dari melakukan tindakan berbahaya tanpa persetujuan dari klien mereka. 

Jika seandainya ragu untuk mengambil tindakan, sebaiknya kita mencari nasihat kepada *Team Lead* atau berdiskusi dengan klien.

## 0x9 - Attack Lifecycle

![Cyber Kill Chain](https://www.lockheedmartin.com/content/dam/lockheed-martin/rms/photo/cyber/THE-CYBER-KILL-CHAIN-body.png.pc-adaptive.1280.medium.png){: w="500" h="400" }
_Cyber Kill Chain_

**Cyber Kill Chain** adalah model yang dikembangkan oleh [Lockheed Martin](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html) untuk menggambarkan tahapan yang biasa dilalui oleh penyerang siber dalam melakukan serangan. 

Model ini membantu para profesional keamanan siber untuk memahami dan mencegah serangan siber.

Berikut adalah tahapan dari *Cyber Kill Chain*:

1. **Reconnaissance:** Penyerang mengumpulkan informasi tentang target mereka. Ini bisa melibatkan segala hal mulai dari OSINT, *scanning*, hingga pengintaian fisik.
    
2. **Weaponization:** Penyerang menciptakan *payload* berbahaya (misalnya, *malware* atau *ransomware*) yang akan digunakan untuk mengeksploitasi target.
    
3. **Delivery:** Penyerang mengirimkan *payload* ke target. Ini bisa dilakukan melalui berbagai metode, termasuk email *phishing*, *drive-by downloads*, atau serangan fisik langsung.
    
4. **Exploitation**: *Payload* mengeksploitasi kerentanan yang ditemukan pada sistem atau jaringan target, sehingga penyerang mendapatkan akses ke sistem.
    
5. **Installation:** Setelah kode berbahaya dieksekusi, penyerang biasanya akan mencoba untuk mempertahankan akses ke sistem dengan menginstal *malware* atau *backdoor* (*persistence*).
    
6. **Command & Control (C2):** Setelah berhasil melakukan *persistence*, *payload* akan mencoba membuat koneksi dengan server C2 penyerang. Ini memberikan penyerang kontrol jarak jauh atas sistem target.
    
7. **Actions on Objectives:** Dengan kontrol atas sistem target, penyerang dapat mulai melaksanakan tujuan mereka. Ini bisa melibatkan segala hal mulai dari pencurian data hingga sabotase sistem.