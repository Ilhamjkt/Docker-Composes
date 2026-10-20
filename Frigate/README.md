# Frigate NVR - Docker Compose

Setup Frigate NVR + Mosquitto (MQTT) yang dipakai di server produksi. Semua kredensial di repo ini adalah dummy, ganti sebelum dipakai.

## Struktur

```
Frigate/
├── docker-compose.yml        # service frigate + mosquitto
├── config/
│   └── config.yml            # konfigurasi Frigate (10 kamera, go2rtc restream)
├── mqtt/
│   └── config/
│       └── mosquitto.conf
└── README.md
```

Folder `storage/` (rekaman) dan file runtime di `config/` (database, `.jwt_secret`, `model_cache/`) dibuat otomatis oleh Frigate saat pertama jalan dan tidak di-commit.

## Kebutuhan

- Docker Engine + Docker Compose v2 (`docker compose`). Panduan instalasi: [../README.md](../README.md)
- Linux host
- IP camera dengan RTSP (config ini memakai path Hikvision `/Streaming/Channels/101`)
- Opsional: iGPU Intel (`/dev/dri`) untuk hwaccel. Hapus bagian `devices` di compose kalau tidak ada.

## Yang harus diganti

### config/config.yml

Di bagian `go2rtc.streams`, ganti tiap URL kamera:

```
rtsp://USERNAME:PASSWORD@CAMERA_1_IP:554/Streaming/Channels/101
```

menjadi username, password, dan IP kamera masing-masing. Kalau password mengandung karakter khusus (`@`, `:`, `/`, `#`), encode dulu (misal `@` jadi `%40`).

Sesuaikan juga sesuai kebutuhan:
- daftar kamera di `cameras`, `camera_groups`, dan `auth.roles`
- `detect` (resolusi/fps)
- `record` (retensi, default 30 hari)

`auth.reset_admin_password: true` membuat password admin di-reset setiap container restart dan password baru ditampilkan di log. Setelah login pertama dan ganti password, ubah ke `false`.

### docker-compose.yml

- `FRIGATE_RTSP_PASSWORD`: ganti dari nilai dummy
- `shm_size`: naikkan kalau kamera banyak / resolusi tinggi
- port: `8971` web UI (HTTPS, dengan login), `8554` RTSP restream, `8555` WebRTC. Port `5000` (tanpa auth) sengaja dimatikan.

### mqtt/config/mosquitto.conf

Default `allow_anonymous true`. Kalau port 1883 terbuka ke luar jaringan lokal, aktifkan autentikasi atau jangan expose port-nya.

## Menjalankan

```bash
cd Frigate
mkdir -p storage
docker compose up -d
docker compose logs -f frigate
```

Ambil password admin awal dari log, lalu buka `https://IP_SERVER:8971`.

## Update

```bash
docker compose pull
docker compose up -d
```

Image memakai tag `stable`, jadi pull bisa menaikkan versi mayor Frigate. Backup folder `config/` sebelum update dan cek release notes Frigate untuk perubahan config.

## Troubleshooting

- `docker compose logs frigate` untuk melihat error
- Cek indentasi YAML
- Pastikan stream kamera bisa dibuka (misal pakai VLC) dengan URL RTSP yang sama
- Pastikan port tidak bentrok dengan service lain
