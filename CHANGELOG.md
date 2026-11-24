# Changelog - Kelompok 3 
Catatan ini merekam setiap pengerjaan, penyesuaian teknis, verifikasi lintas anggota, serta penyempurnaan repositori proyek **Sistem Pemantau Suhu dan Kelembapan Udara Berbasis IoT** yang dikembangkan oleh **Kelompok 3**.

## 23 November 2026 — Audit Silang Tim & Penyelesaian Akhir Repositori Oleh Seluruh Anggota
* **Verifikasi Silang Antaranggota (Cross-Audit):** Setiap anggota Kelompok 3 (**Adzkia Zafira Sahlah**, **Clarissa Daltalena Vaniswa**, **Jhon Calvin Titirloloby**, dan **Arkan Abdurrahman**) melakukan uji coba ulang secara bergantian terhadap dokumentasi serta konfigurasi tahap milik rekan satu tim, untuk memastikan seluruh instruksi reproduksi di `README.md` dapat berjalan tanpa kendala dari awal hingga akhir.
* **Sterilisasi Kredensial:** Mengganti seluruh informasi sensitif (SSID/kata sandi WiFi, kredensial MQTT, dan token API InfluxDB) dengan variabel pengganti (`ISI_SSID_WIFI`, `ISI_PASSWORD_WIFI`, `ISI_PASSWORD_MQTT`, `ISI_TOKEN_INFLUXDB`) sebelum publikasi repositori, melengkapi bukti tiap tahap ke dalam folder docs/, serta finalisasi README.md, CHANGELOG.md, dan kontribusi.md.

---
## Peran B - 23 November 2026 — Penyelesaian Akhir & Sinkronisasi Repositori
* **Penyelarasan Firmware (`firmware/dht22_mqtt.ino`):** Menyeragamkan interval pembacaan sensor DHT22 (`GPIO4`) menjadi 10 detik (`10000 ms`) serta memperbarui identitas klien menjadi `ESP32_Kel3_Client` pada topik `iot/kel3/dht22`.
* **Pembaruan Bukti Pengujian (`docs/`):** Memperbarui arsip tangkapan layar verifikasi lalu lintas broker MQTT (`mosquitto_sub -h localhost -t "iot/kel3/#" -v`) dan kelengkapan bukti pengujian setiap tahap.
* **Sterilisasi Kredensial:** Mengganti seluruh informasi sensitif (SSID/kata sandi WiFi, kredensial MQTT, dan token API InfluxDB) dengan variabel pengganti (`ISI_SSID_WIFI`, `ISI_PASSWORD_WIFI`, `ISI_PASSWORD_MQTT`, `ISI_TOKEN_INFLUXDB`) sebelum publikasi repositori.
* **Kelengkapan Dokumen:** Merampungkan panduan instalasi pada `README.md`, catatan pembagian kerja di `KONTRIBUSI.md`, serta ekspor konfigurasi dashboard pada `dashboard/grafana_kel3.json`.

---

## Peran C - 22 November 2026 — Konfigurasi Akses Publik via ngrok
* **Layanan Tunnel Otomatis (`rpi/ngrok-tunnel.service`):** Menambahkan unit layanan *systemd* (`Restart=always`) pada `rpi-kel3` untuk menjalankan tunnel HTTPS **ngrok** dengan domain statis `headcount-engaged-pushup.ngrok-free.dev` yang mengarah ke antarmuka Grafana (`localhost:3000`).
* **Uji Akses Jaringan Eksternal:** Memverifikasi kelancaran akses dashboard dari luar jaringan lokal kampus menggunakan koneksi data seluler dan memastikan halaman autentikasi login tetap aktif.

---

## Peran D - 20 November 2026 — Basis Data InfluxDB & Dashboard Grafana
* **Penyimpanan Time-Series:** Menyiapkan organisasi `kampus` dan *bucket* `iot` pada **InfluxDB 2.x** (`port 8086`) untuk menampung data historis suhu dan kelembapan secara persisten.
* **Visualisasi & Alerting:** Merancang panel grafik *Time Series* dan indikator *Gauge* pada dashboard **Grafana** (`monitoring IoT kelompok 3`) menggunakan bahasa kueri Flux, sekaligus menambahkan aturan peringatan (*alert*) apabila suhu ruangan melebihi `30°C`.
* **Optimalisasi Kueri:** Menyaring kolom keluaran kueri Flux (`keep(columns: ["_time", "_value", "_field"])`) agar legenda grafik tampil bersih serta memisahkan evaluasi *field* suhu dari kelembapan pada aturan peringatan.

---

## Peran C - 18 November 2026 — Layanan Broker MQTT & Agen Telegraf
* **Broker Pesan Lokal (`rpi/mosquitto.conf`):** Mengaktifkan layanan **Eclipse Mosquitto** pada port `1883` dengan akun klien `esp32` untuk menerima transmisi data dari mikrokontroler.
* **Penghubung Data (`rpi/telegraf.conf`):** Mengonfigurasi agen **Telegraf** untuk berlangganan topik `iot/kel3/dht22`, memproses muatan berformat JSON (`suhu` dan `kelembapan`), serta menuliskannya ke InfluxDB.

---

## Peran B - 12 November 2026 — Perakitan Sensor DHT22 & Firmware ESP32
* **Integrasi Perangkat Keras:** Menghubungkan modul sensor **DHT22** ke papan **ESP32 DevKit** (`VCC` ke `3V3`, `DATA` ke `D4`/`GPIO4`, `GND` ke `GND`).
* **Pemrograman Mikrokontroler:** Mengunggah sketsa program ke ESP32 untuk koneksi jaringan WiFi lokal `2,4 GHz`, indikator LED internal (`GPIO2`) sebagai penanda status koneksi, serta pengiriman paket data JSON secara berkala.

---

## Peran A - 11 November 2026 — Inisialisasi Server Raspberry Pi & Keamanan Jaringan
* **Persiapan Sistem Operasi:** Menginstal **Raspberry Pi OS (64-bit)** dengan *hostname* `rpi-kel3` dan akun `team_3` pada alamat IP lokal `192.168.10.90`.
* **Kendali Jarak Jauh & Firewall:** Mengaktifkan **Raspberry Pi Connect** untuk manajemen server secara *headless* serta menerapkan aturan penyaringan lalu lintas jaringan menggunakan **UFW** pada segmen `192.168.10.0/24`.
### Ditambahkan
- Melakukan instalasi sistem operasi Raspberry Pi OS 64-bit dan menetapkan hostname unik `rpi-kel3`.
- Mengaktifkan firewall UFW (`ufw enable`) untuk membatasi akses port jaringan lokal.
- Mengonfigurasi Raspberry Pi Connect sebagai jalur manajemen terminal dan desktop jarak jauh untuk mengatasi pemblokiran komunikasi langsung antarperangkat (*Client / AP Isolation*) pada jaringan WiFi kampus.
