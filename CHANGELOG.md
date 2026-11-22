# Changelog - Kelompok 3 (Pemantau Suhu dan Kelembapan)

Seluruh riwayat pengembangan, pengujian bertahap, serta perbaikan konfigurasi sistem pemantauan suhu dan kelembapan berbasis ESP32, MQTT, InfluxDB, dan Grafana dicatat dalam dokumen ini.

## [v1.4.0] - 2026-11-22
### Ditambahkan
- Mengonfigurasi layanan latar belakang `systemd` (`ngrok-tunnel.service` dengan `Restart=always`) pada `rpi-kel3` agar tunnel HTTPS ke port `3000` (Grafana) berjalan otomatis saat booting dan tidak terputus ketika sesi terminal ditutup.
- Melakukan pengujian akses dashboard Grafana melalui jaringan data seluler di luar jaringan WiFi kampus (`https://headcount-engaged-pushup.ngrok-free.dev`) dan memverifikasi mekanisme wajib login.
- Melengkapi dokumentasi repositori (`README.md`, `CHANGELOG.md`, `kontribusi.md`, serta arsip bukti pengujian di direktori `docs/`).

## [v1.3.0] - 2026-11-17
### Ditambahkan
- Membuat bucket `iot` pada InfluxDB 2.x (port `8086` lokal) dan memverifikasi persistensi data pasca-*reboot* perangkat Raspberry Pi.
- Membangun dashboard Grafana (port `3000`) menggunakan bahasa query Flux untuk panel grafik *Time Series* dan indikator *Gauge* satuan `°C` dan `%`.
- Menambahkan aturan peringatan (*alert rule*) suhu tinggi dengan kondisi `IS ABOVE 30`.

### Diperbaiki
- **Query Flux Grafik & Gauge:** Menambahkan fungsi `|> keep(columns: ["_time", "_value", "_field"])` untuk menghapus metadata tag bawaan (`host`, `topic`) sehingga teks legenda tidak lagi menabrak angka visual.
- **Filter Query Alert:** Menghapus *field* `kelembapan` dari query evaluasi alert agar peringatan suhu tinggi (`> 30°C`) tidak terpicu secara palsu oleh nilai kelembapan ruang normal (~50%).

## [v1.2.0] - 2026-11-15
### Ditambahkan
- Menginstal dan mengonfigurasi broker Eclipse Mosquitto (`listener 1883`) pada `rpi-kel3`.
- Menerapkan autentikasi kredensial pengguna (`allow_anonymous false`) menggunakan `mosquitto_passwd` untuk mengamankan komunikasi pesan lokal.
- Mengonfigurasi layanan Telegraf (`/etc/telegraf/telegraf.conf`) untuk berlangganan (*subscribe*) ke topik `iot/kel3/dht22` dan menulis payload JSON ke bucket `iot` InfluxDB.

### Diperbaiki
- Menyelaraskan penamaan topik (`iot/kel3/dht22`) dan nama *field* JSON (`suhu` dan `kelembapan`) antara firmware ESP32, Telegraf, dan query Flux Grafana untuk mengatasi status *No data*.

## [v1.1.0] - 2026-11-14
### Ditambahkan
- Merakit rangkaian sensor suhu dan kelembapan DHT22 pada mikrokontroler ESP32 (pin data GPIO 4).
- Mengunggah firmware pembacaan sensor dan pengiriman payload JSON via MQTT setiap 10 detik melalui jaringan WiFi lokal `2,4 GHz`.

### Diperbaiki
- Menyesuaikan interval pembacaan menjadi 10 detik (`10000 ms`) serta menstabilkan koneksi pin GPIO 4 untuk menghilangkan error pembacaan nilai kosong (`nan`).

## [v1.0.0] - 2026-11-12
### Ditambahkan
- Melakukan instalasi sistem operasi Raspberry Pi OS 64-bit dan menetapkan hostname unik `rpi-kel3`.
- Mengaktifkan firewall UFW (`ufw enable`) untuk membatasi akses port jaringan lokal.
- Mengonfigurasi Raspberry Pi Connect sebagai jalur manajemen terminal dan desktop jarak jauh untuk mengatasi pemblokiran komunikasi langsung antarperangkat (*Client / AP Isolation*) pada jaringan WiFi kampus.
