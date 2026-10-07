## Insight untuk SOC

[<- kembali ke write-up The Report](README.md)

Dari hasil membaca laporan, ini yang menurut saya perlu diprioritaskan
oleh SOC yang baru dibentuk.

### 1. Prioritas deteksi

| Prioritas | Area | Dasar dari laporan | Alasan |
|---|---|---|---|
| 1 | Teknik ATT&CK dengan dampak pelanggan terbesar ([ID teknik]) | Daftar top teknik (soal 2) | Teknik yang paling sering dipakai memberi hasil deteksi terbanyak per rule yang dibuat |
| 2 | Eksekusi file skrip berbahaya (JS) | Detection opportunities (soal 6) | Vektor akses awal yang dipicu pengguna sulit dicegah, jadi deteksi di endpoint penting |
| 3 | Eksploitasi kerentanan [Exchange / driver] | Tren kerentanan 2021 (soal 3 dan 4) | Kerentanan yang dieksploitasi aktif perlu dipantau lewat patching dan deteksi |
| 4 | Aktivitas pasca-akses awal oleh afiliasi ransomware | Model afiliasi (soal 7) | Akses awal sering dijual sebelum enkripsi, sehingga masih ada waktu untuk memutus serangan |

### 2. Ide rule deteksi

- **Perilaku yang dideteksi:** eksekusi file skrip (`.js`) yang dijalankan
  oleh proses induk yang tidak lazim.
- **Log source:** event pembuatan proses di endpoint ([Sysmon Event ID 1 /
  Windows Security Event 4688]).
- **Logika singkat:** peringatan muncul jika proses induk adalah
  [nama proses dari temuanmu] dan proses anak menjalankan file `.js`.
- **Catatan:** perlu diuji di lab untuk melihat false positive, misalnya
  dari aplikasi internal yang sah. Rule ini bisa diuji di lab Wazuh.

### 3. Mitigasi yang direkomendasikan

- **Remote Desktop (RDP):** aktifkan [langkah pengamanan dari soal 10]
  dan batasi akses RDP dari internet.
- **Perangkat lunak usang:** inventaris dan patch [perangkat lunak dari
  soal 8], karena menjadi sasaran utama penambang koin.
- **Patch management:** prioritaskan sistem yang terpapar internet,
  terutama [platform dari soal 3].
- **Kesadaran pengguna:** akses awal lewat hasil pencarian (SEO poisoning)
  menunjukkan bahwa pengguna perlu dilatih tentang sumber unduhan.

### 4. Hal yang perlu dibangun SOC lebih dulu

1. Pengumpulan log endpoint yang konsisten (proses, PowerShell, event login).
2. Inventaris aset dan perangkat lunak agar kerentanan bisa dipetakan.
3. Beberapa rule deteksi prioritas dari tabel di atas, lalu diuji dan
   disempurnakan.
4. Pemantauan RDP dan akses dari luar.
