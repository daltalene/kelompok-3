# Kelompok 3 - Pemantau Suhu dan Kelembapan

## Anggota dan peran
| Nama | NIM | Peran |
| :--- | :--- | :--- |
| Adzkia Zafira Sahlah | 125103049 | Peran A - Network dan RPI (Tahap 1 / Koordinator) |
| Clarissa Daltalena Vaniswa | 125103037 | Peran B - Sensor dan Firmware (Tahap 2) |
| Jhon Calvin Titirloloby  | 125103086 | Peran C - Platform dan Akses Internet (Tahap 3a & 3b) |
| Arkan Abdurrahman | 125103038 | Peran D - Data dan Dashboard (Tahap 4) |

## Arsitektur
```text
┌───────────────────┐     WiFi 2,4 GHz      ┌───────────────────────────────────────────────────────────────┐
│   ESP32 + DHT22   │ ────────────────────> │                  Raspberry Pi (rpi-kel3)                    │
│ (Interval 10 dtk) │      MQTT :1883       │                                                               │
└───────────────────┘                       │  ┌───────────┐   ┌──────────┐   ┌────────────┐   ┌─────────┐  │
                                            │  │ Mosquitto │──>│ Telegraf │──>│ InfluxDB 2 │──>│ Grafana │  │
                                            │  │  (:1883)  │   │  (MQTT)  │   │  (:8086)   │   │ (:3000) │  │
                                            │  └───────────┘   └──────────┘   └────────────┘   └────┬────┘  │
                                            └───────────────────────────────────────────────────────┼───────┘
                                                                                                    │
┌───────────────────┐     HTTPS (Publik)    ┌──────────────────┐          Localhost :3000           │
│  Browser Publik   │ <──────────────────── │   ngrok Tunnel   │ <──────────────────────────────────┘
│   (Wajib Login)   │                       │ (Layanan systemd)│
└───────────────────┘                       └──────────────────┘

*Jalur Manajemen Terpisah (Bukan Jalur Data):
[Mahasiswa] <══════> [Raspberry Pi Connect] <══════> [Terminal & Desktop rpi-kel3]
```

Sensor DHT22 pada mikrokontroler ESP32 membaca suhu dan kelembapan udara setiap 10 detik, lalu mengirimkannya dalam format JSON melalui protokol MQTT (port 1883) ke broker Eclipse Mosquitto di Raspberry Pi (`rpi-kel3`). Layanan Telegraf mengambil pesan dari topik `iot/kel3/dht22` dan menyimpannya ke dalam bucket `iot` pada basis data InfluxDB 2.x (port 8086 lokal). Grafana (port 3000) menarik data dari InfluxDB menggunakan bahasa query Flux untuk menampilkan grafik Time Series, indikator Gauge (`°C` dan `%`), serta mengevaluasi aturan peringatan suhu tinggi (`> 30°C`), dan menjadi satu-satunya layanan yang dibuka ke Internet publik melalui layanan tunnel terenkripsi ngrok, sedangkan akses terminal dan desktop RPI oleh mahasiswa dikelola secara terpisah melalui Raspberry Pi Connect.

## Hasil uji per tahap
| Tahap | Bukti | Tanggal |
| :--- | :--- | :--- |
| Tahap 1 - RPI ke Internet | `docs/tahap1_rpi_piconnect.png` (Sesi Pi Connect aktif, `ip a`, `ufw status` aktif, dan `ping 1.1.1.1` berhasil) | 12 Nov 2026 |
| Tahap 2 - Sensor ke RPI | `docs/tahap2_rangkaian_serial.png` (Foto rangkaian ESP32+DHT22 & Serial Monitor pembacaan tiap 10 detik tanpa `nan`) | 17 Nov 2026 |
| Tahap 3a - Broker MQTT | `docs/tahap3a_mosquitto_sub.png` (Tangkapan layar `mosquitto_sub -t 'iot/#'` menerima JSON terautentikasi dari ESP32) | 17 Nov 2026 |
| Tahap 4 - Basisdata dan dashboard | `docs/tahap4_grafana_dashboard.png` (Data tersimpan di bucket `iot` pasca-reboot, grafik, gauge, & alert suhu aktif) | 19 Nov 2026 |
| Tahap 3b - Akses Internet | `docs/tahap3b_akses_seluler.png` (Tangkapan layar akses alamat publik ngrok dari jaringan data seluler & wajib login) | 20 Nov 2026 |

## Akses publik
* **Alamat dashboard:** [https://headcount-engaged-pushup.ngrok-free.dev]
* **Jalur tunnel yang dipakai:** ngrok (`ngrok http --domain=headcount-engaged-pushup.ngrok-free.dev 3000` dijalankan sebagai layanan `systemd`)
* **Video demo (3–5 menit):** `[isi-link-google-drive-atau-youtube-video-demo-kelompok-3]`+

## Kendala dan solusi
| Gejala | Penyebab | Perbaikan |
| :--- | :--- | :--- |
| Akses langsung ke IP lokal RPI (port 3000) gagal dari WiFi kampus | Fitur *Client/AP Isolation* pada WiFi kampus memblokir komunikasi langsung antar-klien lokal | Menggunakan Raspberry Pi Connect untuk manajemen RPI dan tunnel HTTPS ngrok untuk akses publik Grafana |
| Grafik Grafana menampilkan *No data* | Ketidaksesuaian nama topik MQTT dan nama *field* (`temperature` vs `suhu`) | Menyamakan topik menjadi `iot/kel3/dht22` di Telegraf dan memfilter *field* `suhu` serta `kelembapan` di Flux |
| Teks legenda grafik & gauge terlalu panjang menabrak angka visual | InfluxDB menyertakan seluruh tag bawaan (`host`, `topic`) secara otomatis ke dalam label seri | Menambahkan fungsi transformasi Flux `|> keep(columns: ["_time", "_value", "_field"])` |
| Aturan alert suhu langsung memicu status peringatan palsu pada suhu ruang normal | Query alert masih menyertakan *field* `kelembapan` (~50%) sehingga langsung melewati ambang batas `> 30` | Menghapus filter `kelembapan` pada query alert sehingga evaluasi `IS ABOVE 30` hanya untuk *field* `suhu` |

## Cara menjalankan ulang
1. **Siapkan sistem operasi Raspberry Pi OS (64-bit), autentikasi SSH, dan aturan firewall UFW:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo hostnamectl set-hostname rpi-kel3
   sudo apt install -y ufw mosquitto mosquitto-clients

   # Konfigurasi kunci SSH (jalankan ssh-keygen -t ed25519 di klien, lalu daftarkan public key)
   mkdir -p ~/.ssh && chmod 700 ~/.ssh
   echo "<ISI_PUBLIC_KEY_ED25519>" >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys

   # Aktivasi aturan firewall UFW
   sudo ufw allow 22/tcp
   sudo ufw allow 1883/tcp
   sudo ufw allow 3000/tcp
   sudo ufw enable
   sudo ufw status
   ```

2. **Atur kredensial pengguna dan layanan broker Eclipse Mosquitto:**
   ```bash
   sudo mosquitto_passwd -c /etc/mosquitto/pwfile esp32
   sudo cp rpi/mosquitto.conf /etc/mosquitto/conf.d/default.conf
   sudo systemctl enable --now mosquitto
   ```
   
3. **Tambahkan repositori InfluxData, pasang InfluxDB 2.x, Telegraf, dan Grafana:**
   ```bash
   curl -fsSL [https://repos.influxdata.com/influxdata-archive.key](https://repos.influxdata.com/influxdata-archive.key) | gpg --dearmor | sudo tee/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
   sudo apt update && sudo apt install -y influxdb2 telegraf grafana
   sudo systemctl enable --now influxdb telegraf grafana-server
   ```
   
4. **Inisialisasi basis data InfluxDB, buat token API, dan konfigurasi Telegraf:**
   ```bash
   influx setup --username admin --password <ISI_PASSWORD_ADMIN> --org kampus --bucket iot --retention 0 --force
   TOKEN=$(influx auth create --org kampus --all-access --json | grep -o '"token": *"[^"]*"' | cut -d'"' -f4)
   sudo bash -c "cat << 'EOF' > /etc/telegraf/telegraf.conf
   [agent]
     interval = \"5s\"
     flush_interval = \"5s\"

   [[outputs.influxdb_v2]]
     urls = [\"[http://127.0.0.1:8086](http://127.0.0.1:8086)\"]
     token = \"$TOKEN\"
     organization = \"kampus\"
     bucket = \"iot\"

   [[inputs.mqtt_consumer]]
     servers = [\"tcp://127.0.0.1:1883\"]
     topics = [\"iot/kel3/dht22\"]
     username = \"esp32\"
     password = \"<ISI_PASSWORD_MQTT>\"
     data_format = \"json\"
   EOF"
   sudo systemctl restart telegraf
   (Setelah layanan aktif, buka http://192.168.10.90:3000 untuk menghubungkan data source InfluxDB (Flux) dan mengimpor dashboard/grafana_kel3.json).
   ```
   
5. **Jalankan layanan ngrok tunnel untuk publikasi akses internet:**
   ```bash
   ngrok config add-authtoken <ISI_AUTHTOKEN_NGROK>
   sudo cp rpi/ngrok-tunnel.service /etc/systemd/system/ngrok.service
   sudo systemctl daemon-reload
   sudo systemctl enable --now ngrok.service
   ```
   
6. **Program mikrokontroler ESP32 DevKit untuk membaca sensor DHT22:**
   * Buka berkas firmware/dht22_mqtt.ino di Arduino IDE (pastikan pustaka DHT sensor library dan PubSubClient sudah terpasang).
   * Sesuaikan nilai ISI_SSID_WIFI, ISI_PASSWORD_WIFI, serta ISI_PASSWORD_MQTT, pilih papan DOIT ESP32 DEVKIT V1, lalu tekan tombol Upload.
