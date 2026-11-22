# Kontribusi Anggota - Kelompok 3

Dokumen ini merinci pembagian tugas dan kontribusi teknis setiap anggota dalam pengerjaan proyek sistem pemantauan suhu dan kelembapan berbasis IoT.

## Ringkasan Peran
| Nama | NIM | Peran | Fokus Tahap |
| :--- | :--- | :--- | :--- |
| Adzkia Zafira Sahlah | 125103049 | Peran A - Network dan RPI (Koordinator) | Tahap 1 |
| Clarissa Daltalena Vaniswa | 125103037 | Peran B - Sensor dan Firmware | Tahap 2 |
| Jhon Calvin Titirloloby | 125103086 | Peran C - Platform dan Akses Internet | Tahap 3a & 3b |
| Arkan Abdurrahman | 125103038 |Peran D - Data dan Dashboard | Tahap 4 |

---

## Rincian Kontribusi per Anggota

### 1. Adzkia Zafira Sahlah (125103049) — Peran A: Network dan RPI (Koordinator)
* **Persiapan Sistem & Jaringan RPI:** Melakukan instalasi Raspberry Pi OS 64-bit, konfigurasi hostname `rpi-kel3`, pembaruan paket sistem, serta verifikasi konektivitas jaringan lokal dan Internet.
* **Keamanan Firewall:** Mengaktifkan dan mengatur kebijakan firewall UFW (`ufw status`) pada Raspberry Pi.
* **Manajemen Akses Jarak Jauh:** Mengonfigurasi Raspberry Pi Connect sebagai jalur manajemen terminal/desktop untuk mengatasi kendala *Client / AP Isolation* pada jaringan WiFi kampus.
* **Koordinasi Proyek:** Mengoordinasikan integrasi antartahap dan pengumpulan dokumentasi bukti uji (`docs/tahap1_rpi_piconnect.png`).

### 2. Clarissa Daltalena Vaniswa (125103037) — Peran B: Sensor dan Firmware
* **Perakitan Perangkat Keras:** Merancang dan merakit rangkaian mikrokontroler ESP32 dengan sensor DHT22 pada pin GPIO 4.
* **Pengembangan Firmware:** Menulis kode program Arduino untuk koneksi WiFi `2,4 GHz`, pembacaan sensor suhu dan kelembapan setiap 10 detik, serta pemformatan payload JSON (`suhu` dan `kelembapan`).
* **Kalibrasi & Validasi Sensor:** Mengatasi kendala pembacaan nilai `nan` dan memverifikasi kestabilan pengiriman data melalui Serial Monitor (`docs/tahap2_rangkaian_serial.png`).

### 3. Jhon Calvin Titirloloby (125103086) — Peran C: Platform dan Akses Internet
* **Konfigurasi Broker MQTT:** Menginstal Eclipse Mosquitto pada `rpi-kel3`, mengatur `listener 1883`, menonaktifkan akses anonim, serta mengelola kredensial autentikasi klien ESP32 (`docs/tahap3a_mosquitto_sub.png`).
* **Konfigurasi Tunnel Publik:** Menyiapkan domain statis ngrok dan membuat layanan permanen `systemd` (`ngrok-tunnel.service`) agar port `3000` Grafana dapat diakses secara aman melalui HTTPS.
* **Pengujian Akses Luar Kampus:** Melakukan pengujian akses publik dashboard Grafana menggunakan jaringan data seluler dan memverifikasi proteksi login (`docs/tahap3b_akses_seluler.png`), serta menyusun dokumentasi repositori GitHub.

### 4. Arkan Abdurrahman (125103038) — Peran D: Data dan Dashboard
* **Konfigurasi Pipeline Data:** Mengonfigurasi InfluxDB 2.x (pembuatan organisasi, bucket `iot`, dan token akses) serta menyelaraskan konfigurasi Telegraf pada topik `iot/kel3/dht22`.
* **Perancangan Visualisasi Grafana:** Membangun panel grafik *Time Series* dan indikator *Gauge* (`°C` dan `%`) menggunakan bahasa query Flux, serta merapikan tampilan legenda dengan fungsi `keep()`.
* **Konfigurasi Alert & Uji Persistensi:** Menyusun aturan peringatan suhu tinggi (`> 30°C`), memperbaiki filter query alert agar akurat hanya pada *field* `suhu`, serta memverifikasi ketahanan data InfluxDB pasca-*reboot* (`docs/tahap4_grafana_dashboard.png`).
