---
title: '03 - Red Team Engagement Planning'
date: 2023-07-08 15:50:22 +0700
categories: ['Red Team', 'Fundamental']
tags: [red team, engagement]     # TAG names should always be lowercase
author: nairpaa
---

> *Proper planning and preparation prevents poor performance*.

Pada kesempatan kali ini, kita akan mempelajari hal-hal yang perlu disiapkan sebelum melakukan *Red Teaming*.

*Engagement Planning* adalah tahap awal dari setiap operasi *Red Teaming*, dan sangat penting untuk keberhasilan seluruh operasi.

Tahap ini mencakup persiapan dan perencanaan yang diperlukan sebelum *Red Teamer* dapat mulai melakukan aktivitas mereka.

Ada beberapa aspek yang harus dipertimbangkan dalam merencanakan sebuah aktivitas *Red Teaming*. Berikut penjelasan singkatnya:

## 0x1 - Scoping

*Scoping* merujuk pada proses mendefinisikan lingkup dan tujuan dari penugasan *Red Teaming*. 

Ini mencakup apa yang harus diuji, di mana, dan kapan. Penting untuk mendefinisikan apa yang dianggap di luar batas sehingga *Red Teaming* tidak melangkah di luar lingkup yang disepakati.

*Red Teaming* biasanya berfokus untuk mencapai tujuan tertentu, bukan melakukan penilaian terhadap semua host dalam suatu jaringan.

Membuat dan menentukan tujuan dari *Red Team Engagement* bisa menjadi sulit, terutama untuk organisasi yang baru mengenal *Red Teaming*. 

Tujuan yang solid harus diputuskan untuk mendapatkan *Red Team Engagement* yang sukses.

> Kita bisa membaca artikel [Red Team Engagement Goal Planning](https://redteam.guide/docs/Planning/goal-planning/) untuk membantu merumuskan tujuan yang spesifik dan relevan untuk organisasi.
{: .prompt-tip }

## 0x2 - Threat Model

Pada tahap ini, *Red Teamer* menyusun model ancaman yang realistis dan relevan terhadap organisasi sesuai *scope* yang ditentukan sebelumnya.

Model ini akan digunakan oleh *Red Team* sebagai metode serangan (TTPs) yang akan dilakukan.

Organisasi yang lebih matang biasanya sudah memiliki gambaran tentang ancaman mereka berdasarkan peristiwa masa lalu, *threat intel*, atau *market reports*. 

Namun, organisasi lain mungkin tidak. Biasanya, organisasi yang tidak memiliki pemahaman yang kuat tentang ancaman mereka cenderung mengasumsikan ancaman paling parah, seperti APT, yang mungkin tidak selalu realistis.

Sebagai bagian dari perencanaan ini, bersiaplah untuk memoderasi atau membantu menyelaraskan harapan mereka.

> Pembahasan tentang cara mempelajari TTPs telah kita bahas sebelumnya pada bagian [How to learn about TTPs?](/posts/what-is-red-teaming/#0x3---how-to-learn-about-ttps).
{: .prompt-info }

## 0x3 - Breach Model

*Breach model* menjelaskan metode yang akan digunakan oleh *Red Team* untuk mendapatkan akses ke lingkungan target. 

Biasanya, ini dilakukan dengan mencoba mendapatkan akses sesuai dengan ancaman yang telah diidentifikasi (misalnya melalui OSINT dan *phishing*), atau akses yang diberikan oleh organisasi itu sendiri (sering disebut "***assume breach***" atau "***ceded access***").

Ada keuntungan dan kerugian untuk setiap pendekatan, tergantung pada tujuan evaluasi.

Jika *assume breach* tidak dipilih dan *Red Team* mencoba mendapatkan akses, penting untuk memiliki rencana cadangan jika akses tidak diperoleh dalam periode waktu yang ditentukan sebelumnya. 

Salah satu opsinya bisa jadi beralih ke model *assume breach* jika *Red Team* belum mendapatkan akses dalam 25% pertama dari durasi penugasan.

Hal ini penting karena penilaian *Red Teaming* lebih tentang deteksi dan respons, daripada pencegahan. 

Sehingga bagian-bagian tersebut lebih penting daripada berusaha "membuktikan" bahwa pelanggaran dapat terjadi di tempat pertama.

Dengan kata lain, tujuan utamanya bukanlah untuk menunjukkan bahwa sistem dapat ditembus, melainkan untuk mengevaluasi sejauh mana organisasi dapat mendeteksi dan merespons ancaman yang ada. 

Oleh karena itu, jika *Red Team* tidak berhasil mendapatkan akses dalam jangka waktu yang telah ditentukan, mereka mungkin akan beralih ke model *assume breach* untuk memastikan bahwa bagian deteksi dan respons dari penilaian tetap dapat dilakukan.

## 0x4 - Notifications & Announcements

*Assessment* jenis ini seringkali diatur oleh *top level management* dalam *role* *security* atau *compliance*, dan mereka menghadapi pilihan apakah mereka akan tidak memberitahu siapa pun, memberitahu semua orang, atau hanya tim *security*/*support* yang relevan terkait *Red Teaming Engagement* ini.

Tidak memberikan pemberitahuan memungkinkan semua orang bereaksi seperti biasa dan akan menghasilkan hasil yang paling otentik. 

Namun, tim *security* mungkin merasa seolah-olah mereka sedang diuji atau tidak dipercaya oleh manajemen, yang dapat menyebabkan hubungan yang buruk dan berdampak negatif pada hasilnya. 

Di sisi lain, memiliki pemberitahuan sebelumnya dapat membuat mereka lebih waspada atau untuk (sementara) meningkatkan langkah-langkah keamanan, yang bukan merupakan refleksi akurat dari postur keamanan sehari-hari mereka.

Akhirnya, keputusan yang "benar" harus didasarkan pada budaya dan hubungan yang ada dalam organisasi. 

Dengan kata lain, pilihan apakah memberi tahu atau tidak tergantung pada dinamika dan kebijakan internal organisasi tersebut. 

Misalnya, dalam organisasi di mana kepercayaan dan transparansi adalah nilai utama, pemberitahuan mungkin menjadi norma. 

## 0x5 - Rules of Engagement (RoE)

*Rules of Engagement (RoE)* mendefinisikan aturan dan metodologi yang akan digunakan dalam *Red Teaming*; dan harus disepakati dan ditandatangani oleh semua pihak. 

RoE seharusnya:
1. Mendefinisikan tujuan *engagement*.
2. Mendefinisikan *scope* *engagement*.
3. Mengidentifikasi persyaratan hukum atau regulasi dan/atau *restrictions*.
4. Berisi daftar kontak darurat untuk orang-orang kunci di semua pihak.

Perubahan apa pun yang dibuat pada RoE juga harus disepakati dan ditandatangani oleh semua pihak yang relevan.

Meskipun *Physical Red Teaming* berada di luar *scope*, anggota dari *engagement* tersebut harus membawa surat izin (*get out of jail letter*), yang ditandatangani dan diotorisasi oleh klien. 

Jika tim tertangkap dan ditahan oleh penegak hukum, mereka perlu membuktikan bahwa mereka bertindak dengan izin untuk menghindari tuntutan hukum.

Dengan kata lain, RoE berfungsi sebagai kontrak atau perjanjian antara *Red Team* dan organisasi yang ditargetkan. 

Tujuannya adalah untuk memastikan bahwa semua pihak memahami tujuan dan metode keterlibatan, dan untuk melindungi *Red Team* dan organisasi dari konsekuensi hukum yang tidak diinginkan.

> Kita bisa melihat contoh [ROE Template](https://redteam.guide/docs/Templates/roe_template/) sebagai referensi.
{: .prompt-tip }

## 0x6 - Record Keeping & Deconfliction

Selama *engagement*, anggota *Red Team* harus menjaga catatan aktivitas mereka. 

Jika insiden terjadi nanti, organisasi dan *Digital Forensics & Incident Response (DFIR)* akan mengidentifikasi apakah aktivitas yang diamati adalah hasil dari *engagement* atau tidak. 

*Framework* seperti Cobalt Strike mencatat semua aktivitas yang dilakukan oleh *Red Team*. Ini termasuk perintah yang dijalankan, alamat IP target dan nama host, *payload*, dan lain-lain. 

Namun, kita juga mungkin menjalankan *tool* selain Cobal Strike yang perlu dicatat secara terpisah.

Jika memungkinkan, catat detail tanggal dan waktu *tool* dijalankan dalam lingkungan target.

> Kita bisa melihat contoh [Operator Log](https://redteam.guide/docs/Templates/oplog/) sebagai referensi.
{: .prompt-tip }

## 0x7 - Data Handling

*Red Team* harus memastikan bahwa mereka mematuhi semua persyaratan organisasional, regulasi dan/atau hukum untuk penanganan data. 

*Red Team* juga bisa menemui data yang bersifat sensitif seperti data pribadi, medis, keuangan yang harus diperlakukan sebagai rahasia. Jika memungkinkan, hindari melihat data ini kecuali itu bagian dari ruang lingkup dan tujuan yang disepakati; dan jangan pernah mempengaruhi kerahasiaan, integritas, atau ketersediaan data.

Ketika *engagement* selesai, data sensitif yang tersimpan oleh penyerang tersebut harus dihancurkan dengan cara yang tepat.

Dengan kata lain, *Red Team* harus memiliki prosedur dan protokol yang ketat untuk penanganan data selama keterlibatan mereka, termasuk bagaimana mereka mengakses, menyimpan, memproses, dan menghancurkan data. 

*Red Team* juga harus memastikan bahwa mereka tidak melanggar hak privasi individu atau hukum perlindungan data selama proses ini.

## 0x8 - Duration

Durasi dari suatu *engagement* hanya harus ditentukan setelah ruang lingkup dan tujuan telah disepakati. 

Ini memungkinkan kita memberikan perkiraan berdasarkan pekerjaan aktual yang telah disepakati. 

Durasi suatu *Red Teaming Engagement* harus ditentukan secara strategis dan didasarkan pada pekerjaan yang harus dilakukan untuk mencapai tujuan yang telah ditetapkan.

Penting juga untuk dicatat bahwa durasi *engagement* mungkin perlu diubah jika ruang lingkup atau tujuan berubah sepanjang jalan, atau jika *Red Team* menemui hambatan atau tantangan yang tidak terduga selama prosesnya. 

Oleh karena itu, fleksibilitas dan penyesuaian adalah kunci dalam menentukan dan mengelola durasi *Red Teaming Engagement*.

## 0x9 - Costs

Ketika menghitung biaya suatu *engagement*, ada banyak faktor yang harus dipertimbangkan.

Sebuah tim harus memiliki setidaknya dua anggota, dan selalu setidaknya satu *lead*. Jumlah anggota tim harus mencerminkan ukuran *engagement* dan jangka waktu penyelesaiannya. Empat anggota (tiga operator dan satu *lead*) adalah ukuran rata-rata tim.

Jika diperlukan tim untuk bepergian (misalnya, jika mereka perlu datang ke lokasi untuk meniru ancaman dari dalam), maka biaya tersebut dan biaya insidental lainnya harus diperhitungkan.

Kebanyakan *Red Team* menggunakan *tools* komersial untuk membantu melaksanakan *engagement* mereka. Model lisensi Cobalt Strike adalah per-operator, jadi jika tim besar diperlukan untuk menyelesaikan *engagement* dalam jangka waktu yang disepakati, mungkin diperlukan lisensi tambahan. 

*Red Team* mungkin juga perlu menggunakan *software* khusus untuk *engagement* tersebut. Banyak *Red Team* menggunakan *cloud* publik untuk menjalankan sebagian dari infrastruktur mereka.

Mereka mungkin juga ingin membeli nama domain untuk digunakan dalam *phishing campaign* atau C2 *traffic*. 

Tim server harus dijalankan di tempat *Red Team* sehingga biaya menjalankan dan memelihara infrastruktur tersebut juga harus dimasukkan.

Sering kali kita lupa untuk menghitung biaya aktivitas yang dilakukan sebelum dan setelah suatu *engagement*. Itu termasuk seluruh proses *engagement* ini, *pre-engagement meetings*, *threat profiling*, *research*, penyesuaian *tools*, infrastruktur, dan lainnya.