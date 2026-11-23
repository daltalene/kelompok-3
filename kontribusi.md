# Catatan Pembagian Tugas & Kontribusi Tim — Kelompok 3

Berkas ini mendokumentasikan pembagian fokus pekerjaan, ruang lingkup konfigurasi teknis, serta tanggung jawab pengelolaan repositori GitHub bagi setiap anggota **Kelompok 3** dalam pengembangan sistem pemantauan suhu dan kelembapan berbasis IoT.

---

## Matriks Tanggung Jawab Anggota

| Nama | NIM | Peran | Fokus Tahap | Artefak & Direktori yang Dikelola |
| :--- | :--- | :--- | :--- | :--- |
| Adzkia Zafira Sahlah | 125103049 | Peran A | Infrastruktur Server & Keamanan Jaringan | `docs/tahap1_piconnect_ufw.png`, `KONTRIBUSI.md` |
| Clarissa Daltalena Vaniswa | 125103037 | Peran B | Perakitan Perangkat Keras & Pemrograman ESP32 | `firmware/dht22_mqtt.ino`, `docs/tahap2_rangkaian_serial.png` |
| Jhon Calvin Titirloloby | 125103086 | Peran C | Layanan Broker MQTT, Agen Data & Tunnel Publik | `rpi/mosquitto.conf`, `rpi/telegraf.conf`, `rpi/ngrok-tunnel.service`, `docs/tahap3a_mosquitto_sub.png`, `docs/tahap3b_akses_publik.png` |
| Arkan Abdurrahman | 125103038 | Peran D | Penyimpanan Time-Series, Visualisasi & Dokumentasi | `dashboard/grafana_kel3.json`, `docs/tahap4_grafana_dashboard.png`, `README.md`, `CHANGELOG.md` |

---

## Detail Realisasi Pekerjaan

### 1. Peran A — Adzkia Zafira Sahlah (Infrastruktur Server & Keamanan Jaringan)
- Menyiapkan sistem operasi **Raspberry Pi OS (64-bit)** sebagai server lokal dengan identitas *hostname* `rpi-kel3` dan user `team_3` pada alamat IP `192.168.10.90`.
- Mengaktifkan akses kendali jarak jauh melalui **Raspberry Pi Connect** guna mempermudah pemeliharaan perangkat di laboratorium tanpa monitor eksternal.
- Menerapkan penyaringan lalu lintas jaringan menggunakan **UFW (*Uncomplicated Firewall*)** agar komunikasi layanan hanya terbuka pada segmen jaringan lokal (`192.168.10.0/24`).
- Mendokumentasikan hasil aktivasi akses jarak jauh dan aturan *firewall* ke dalam `docs/tahap1_piconnect_ufw.png` serta menyusun catatan kontribusi tim.

### 2. Peran B — Clarissa Daltalena Vaniswa (Perakitan Perangkat Keras & Pemrograman ESP32)
- Menghubungkan modul sensor **DHT22** pada papan mikrokontroler **ESP32 DevKit** menggunakan jalur daya `3V3`, pin sinyal digital `D4` (`GPIO4`), dan referensi `GND`.
- Menulis logika program `firmware/dht22_mqtt.ino` yang mengatur siklus pembacaan suhu dan kelembapan setiap 10 detik, menyalakan indikator LED internal (`GPIO2`) saat koneksi stabil, serta mengirimkan data berformat JSON ke topik `iot/kel3/dht22`.
- Memvalidasi keluaran pembacaan sensor pada *Serial Monitor* (baud rate `115200`) dan menyimpan bukti pengujian perangkat keras pada `docs/tahap2_rangkaian_serial.png`.

### 3. Peran C — Jhon Calvin Titirloloby (Layanan Broker MQTT, Agen Data & Tunnel Publik)
- Mengelola layanan **Eclipse Mosquitto** melalui berkas `rpi/mosquitto.conf` untuk menangani pertukaran pesan MQTT lokal dengan kredensial klien `esp32`.
- Menjalankan **Telegraf** (`rpi/telegraf.conf`) sebagai penghubung yang menerjemahkan muatan JSON dari broker MQTT ke dalam basis data InfluxDB.
- Membuat unit layanan *systemd* permanen (`rpi/ngrok-tunnel.service`) berbasis **ngrok** dengan domain statis `headcount-engaged-pushup.ngrok-free.dev` agar antarmuka Grafana lokal (`port 3000`) dapat diakses dari internet.
- Mengarsipkan tangkapan layar verifikasi lalu lintas topik MQTT (`docs/tahap3a_mosquitto_sub.png`) dan pengujian akses luar jaringan (`docs/tahap3b_akses_publik.png`).

### 4. Peran D — Arkan Abdurrahman (Penyimpanan Time-Series, Visualisasi & Dokumentasi)
- Membangun struktur penyimpanan metrik pada **InfluxDB 2.x** di bawah organisasi `kampus` dan *bucket* `iot`, lengkap dengan pengaturan token otorisasi API.
- Menyusun tampilan pemantauan *real-time* di **Grafana** bertajuk *monitoring IoT kelompok 3* (memuat grafik deret waktu dan indikator *gauge* suhu-kelembapan), lalu mengekspor konfigurasinya ke `dashboard/grafana_kel3.json`.
- Mengambil tangkapan layar dashboard yang telah menerima aliran data aktual untuk disimpan pada `docs/tahap4_grafana_dashboard.png`.
- Merapikan struktur repositori, melengkapi panduan instalasi serta tabel verifikasi pada `README.md`, dan mencatat riwayat pembaruan proyek di `CHANGELOG.md`.
