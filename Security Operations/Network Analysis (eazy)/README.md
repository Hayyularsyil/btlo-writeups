\# Network Analysis - Blue Team Labs Online (Write-up)

&#x20;

\## Scenario

SOC menerima peringatan di SIEM mereka untuk 'Pemindaian Port Lokal ke Lokal' di mana IP pribadi internal mulai memindai sistem internal lainnya. Dapatkah Anda menyelidiki dan menentukan apakah aktivitas ini berbahaya atau tidak? Anda telah diberikan PCAP, selidiki menggunakan alat apa pun yang Anda inginkan.



\## Methodology

Setelah file zip diekstrak saya mulai buka file `.pcap` nya via Microsoft Edge. Nah LAB ini fokusnya adalah analis terhadap laporan atau peringatan SIEM untuk menganalisis file yang .pcap yang diberikan apakah terdapat aktivitas berbahaya atau tidak.



\## Tools

1\. Virtual Machine (Kali Linux)

2\. Wireshark



!\[](image/wireshark.png)



\*\*IP mana yang bertanggung jawab untuk melakukan aktivitas pemindaian port?\*\*



Pada bagian ini kita ingin mencari IP yang melakukan aktivitas pemindaian port. Langkah yang kita akan kita lakukan di wireshark adalah buka bagian tab **Statistik --> Conversation -->**  setelah masuk di bagian conversation kita bisa klik tab **IPv4** untuk mengurutkan jumlah paket dan kita akan menemukan IP penyerang akan menjadi pengirim dengan jumlah paket SYN/TCP yang jauh lebih banyak dibandingkan dengan host/ip yang lain.



!\[](image/image1.png)



\*\*Rentang port apa yang dipindai oleh host yang mencurigakan?\*\*



Pada bagian kita akan mencari tahu rentang port yang di Scan oleh si penyerang. Caranya kita akan memasukan filter untuk memfilter trafik penyerang berikut filternya **ip.src == <IP-Penyerang> \&\& tcp.flag.syn == 1** makan akan muncul paket-paket hasil filternya di layar wireshark. namun kita tidak mencari disana kita akan cari dengan membuka tab **statistic --> Conversation** setelah masuk di layer Conversation kita klik tab **Port** untuk mengurutkan port dari yang terkecil yang terbesar itulah rentang port yang di scan oleh penyerang.



!\[](image/image2.png)



\*\*Jenis pemindaian port apa yang dilakukan?\*\*



Pada bagian ini kita akan mengamati paket TCP yang dikirim penyerang saat melakukan scanning. Kita bisa memasukan filter tcp.flag.syn == 1 \&\& tcp.flag.ack == 0 . Jika setelah kita filter dengan filter tersebut dan kemudian paket yang muncul hanya SYN tanpa menyelesaikan 3 way handshake maka ia disebut  **TCP SYN**, namun jika ia menyelesaikan 3 way handshake (SYN SYN-ACK RST) maka disebut **TCP Connect Scan**.



!\[](image/image01.png)



\*\*Dua tools lagi digunakan untuk melakukan pengintaian terhadap Open Port, apa saja tools tersebut?\*\*



Pada bagian ini spesifik kita diminta untuk mencari tahu mengenai tools yang digunakan penyerang untuk Rekognisi/Enumerasi. Pada bagian ini saya menggunakan filter **http.request** untuk melihat paket request dari si penyerang tool-tool ini biasanya meninggalkan jejak. kita bisa coba buka salah satu paket kemudian cek tab dibawahnya spesifik kolom detail http di bagian user-agent.



!\[](image/image3.png)

!\[](image/image4.png)



\*\*Apa nama file PHP yang digunakan penyerang untuk mengunggah web shell? \*\*



Pada bagian kita diminta untuk menemukan file php yang digunakan penyerang untuk mengunggah web shell. Nah caranya disini kita masukan filter Kembali dengan **http.request.method == "POST"** kemudian buka salah satu paket klik kanan lalu pilih **Follow --> HTTP Stream** nanti akan muncul jendela berupa teks merah untuk request dan teks biru untuk response scroll ke Bawah untuk menemukan endpoind upload pada bagian teks biru (response).



!\[](image/image5.png)



\*\*Apa nama web shell yang diunggah oleh penyerang?\*\*



Pada bagian ini kita diminta menganalis bagian nama web shell yang diunggah atau upload oleh penyerang. Tetap gunakan filter pada no 5 **http.request.method == "POST"** kemudian cara paling gampang cek kolom info kemudian scroll sampai menemukan info upload kemudian klik kanan pilih **Follow --> HTTP Stream** setelah muncul jendela lalu amati dibagian Content-Disposition.



!\[](image/image7.png)





\*\*Parameter apa yang digunakan di web shell untuk mengeksekusi perintah?\*\*



Jangan tutup jendela HTTP Stream pada packet tadi. Selanjutnya kita akan amati dibagian kode php nya lihat dan amati parameter apa yang digunakan amati dengan seksama pada tanda kurung siku setelah \_REQUEST.



!\[](image/image8.png)





\*\*Apa perintah pertama yang dieksekusi oleh penyerang?\*\*



Pada bagian ini kita akan mencari perintah yang dieksekusi pertama oleh penyerang, sebagai seorang analisis keamanan tentu kita harus tahu eksekusi pertama yang dilakukan oleh penyerang untuk memetakan apa yang dijalankan, sistem apa yang ditargetkan ataupun data apa yang akan dicuri dengan eksekusinya tersebut. caranya adalah dengan memanfaatkan parameter yang kita temukan tadi dengan menggunakan filter **http.request.uri contains "cmd="** nah kita bisa amati di salah satu paket dibagian kolom info setelah parameter cmd.



!\[](image/image9.png)



\*\*Jenis koneksi shell apa yang diperoleh penyerang melalui eksekusi perintah?\*\*



Masih di layar yang sama dengan dengan sebelumnya kita amati dibagian kolom info ekskusi perintah yang dijalankan kemudian klik paket tersebut dan kita amati di bagian tab bawah tentang paket atau bisa menggunakan HTTP Stream. Jika penyerang 

mengeksekusi perintah bash/python yang dimana membuat victim menghubungi kembali si penyerang maka tipe koneksi shell tersebut adalah **Reverse Shell** .

!\[](image/image11.png)



\*\*Port apa yang dia gunakan untuk koneksi shell?\*\*



Masih pada paket tadi, kalau tadi saya hanya mengandalkan keterangan di tab bagian bawah selanjutnya untuk menemukan port untuk koneksi shell oleh penyerang saya membuka paket tadi dengan HTTP Stream kemudian kita amati atau perhatikan dengan seksama dibagian s.connect disitu dengan ip dan port.



!\[](image/image12.png)



\## Referensi

\- \[Lab Network Analysis](https://blueteamlabs.online/home/challenge/network-analysis-web-shell-d4d3a2821b)

\- \[File pcap](https://blueteamlabs.online/storage/files/DdMgCqiLCvTxMd6XYQgaXCyjDH8M6b.zip)

