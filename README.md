# Kelompok 3 - Pemantau Suhu dan Kelembapan

## Anggota dan peran
| Nama | NIM | Peran |
| :--- | :--- | :--- |
| Adzkia Zafira Sahlah | 125103049 | Peran A - Network dan RPI (Tahap 1 / Koordinator) |
| Clarissa Daltalena Vaniswa | 125103037 | Peran B - Sensor dan Firmware (Tahap 2) |
| Jhon Calvin Titirloloby  | 125103086 | Peran C - Platform dan Akses Internet (Tahap 3a & 3b) |
| Arkan Abdurrahman | 125103038 | Peran D - Data dan Dashboard (Tahap 4) |

## Arsitektur
```mermaid
flowchart LR
    subgraph Node["Node Sensor"]
        ESP["ESP32 + DHT22<br/>(Interval 10 dtk)"]
    end

    subgraph RPI["Raspberry Pi (rpi-kel3)"]
        MQTT["Mosquitto Broker<br/>(Port 1883)"]
        TEL["Telegraf<br/>(Topic: iot/kel3/dht22)"]
        INF["InfluxDB 2.x<br/>(Port 8086 | Bucket: iot)"]
        GRAF["Grafana<br/>(Port 3000)"]
    end

    subgraph Publik["Akses Publik"]
        TUN["ngrok Tunnel<br/>(HTTPS)"]
        USR["Browser Publik<br/>(Wajib Login)"]
    end

    subgraph Mgmt["Jalur Manajemen Terpisah"]
        MHS["Mahasiswa"] <--> PIC["Raspberry Pi Connect<br/>(Terminal & Desktop)"]
    end

    ESP -- "WiFi 2,4 GHz / MQTT" --> MQTT
    MQTT -- "Subscribe" --> TEL
    TEL -- "Write" --> INF
    INF -- "Flux Query" --> GRAF
    GRAF -- "Localhost:3000" --> TUN
    TUN -- "HTTPS" --> USR
    PIC -. "Kontrol Sistem" .-> RPI
    ```Sensor DHT22 pada mikrokontroler...

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
```bash
# 1. Perbarui sistem operasi Raspberry Pi OS 64-bit dan atur hostname unik
sudo apt update && sudo apt full-upgrade -y
sudo hostnamectl set-hostname rpi-kel3

# 2. Pasang dan aktifkan firewall UFW serta layanan inti (Mosquitto, InfluxDB, Telegraf, Grafana)
sudo apt install -y ufw mosquitto mosquitto-clients influxdb2 telegraf grafana
sudo ufw enable
sudo systemctl enable --now mosquitto influxdb telegraf grafana-server

# 3. Verifikasi aliran pesan MQTT lokal terautentikasi dari ESP32
mosquitto_sub -h localhost -p 1883 -u <mqtt_user> -P <mqtt_password> -t 'iot/kel3/dht22' -v

# 4. Jalankan layanan systemd tunnel ngrok untuk mengekspos port 3000 (Grafana) ke Internet
sudo systemctl enable --now ngrok-tunnel.service
