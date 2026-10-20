# Frigate NVR - Docker Compose

Frigate NVR + Mosquitto (MQTT) setup as used on a production server. All credentials in this repo are placeholders; replace them before deploying.

## Structure

```
Frigate/
  docker-compose.yml        # frigate + mosquitto services
  config/
    config.yml              # Frigate config (10 cameras, go2rtc restream)
      mqtt/
      config/
          mosquitto.conf
       README.md
```

The `storage/` folder (recordings) and runtime files in `config/` (database, `.jwt_secret`, `model_cache/`) are created by Frigate on first start and are not committed.

## Requirements

- Docker Engine + Docker Compose v2 (`docker compose`). Install guide: [../README.md](../README.md)
- Linux host
- IP cameras with RTSP (this config uses the Hikvision path `/Streaming/Channels/101`)
- Optional: Intel iGPU (`/dev/dri`) for hwaccel. Remove the `devices` section in the compose file if you don't have one.

## What to change

### config/config.yml

Under `go2rtc.streams`, replace each camera URL:

```
rtsp://USERNAME:PASSWORD@CAMERA_1_IP:554/Streaming/Channels/101
```

with that camera's username, password and IP. If a password contains special characters (`@`, `:`, `/`, `#`), URL-encode them (e.g. `@` becomes `%40`).

Adjust as needed:
- camera list in `cameras`, `camera_groups` and `auth.roles`
- `detect` (resolution/fps)
- `record` (retention, 30 days by default)

`auth.reset_admin_password: true` resets the admin password on every container restart and prints the new one in the logs. Set it to `false` after your first login and password change.

### docker-compose.yml

- `FRIGATE_RTSP_PASSWORD`: replace the placeholder value
- `shm_size`: increase for many cameras / high resolution
- ports: `8971` web UI (HTTPS, with login), `8554` RTSP restream, `8555` WebRTC. Port `5000` (no auth) is disabled on purpose.

### mqtt/config/mosquitto.conf

Defaults to `allow_anonymous true`. If port 1883 is reachable from outside your local network, enable authentication or don't expose the port.

## Run

```bash
cd Frigate
mkdir -p storage
docker compose up -d
docker compose logs -f frigate
```

Get the initial admin password from the logs, then open `https://SERVER_IP:8971`.

## Update

```bash
docker compose pull
docker compose up -d
```

The image uses the `stable` tag, so a pull can bring a new major Frigate version. Back up `config/` before updating and check the Frigate release notes for config changes.

## Troubleshooting

- `docker compose logs frigate` to see errors
- Check YAML indentation
- Make sure the camera stream opens (e.g. in VLC) with the same RTSP URL
- Make sure the ports don't conflict with other services
