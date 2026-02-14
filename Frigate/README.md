# Frigate NVR - Docker Compose

A complete NVR designed for Home Assistant with AI object detection. Frigate uses OpenCV and Tensorflow to perform realtime object detection locally for IP cameras.

⚠️ **Warning**: This guide assumes you're using **root** privileges. Use at your own risk.

---

## Features

- 🎯 Real-time object detection
- 🎥 24/7 recording with smart retention
- 🔔 Intelligent notifications
- 🏠 Home Assistant integration
- 📱 Mobile-friendly web interface
- 🚀 Hardware acceleration support (VAAPI, CUDA, OpenVINO)
- 📊 Built-in statistics and timeline

---

## Prerequisites

- Docker and Docker Compose installed
- IP Camera(s) with RTSP support
- Sufficient storage for recordings
- (Optional) Google Coral TPU or GPU for better performance

---

## Quick Start

### 1. Clone or Download

```bash
git clone (https://github.com/Ilhamjkt/Docker-Composes.git)
cd frigate
```

### 2. Create Required Directories

```bash
mkdir -p config
mkdir -p storage
mkdir -p media/frigate
```

### 3. Configure Frigate

Edit `config/config.yml` with your camera settings (see configuration examples below).

### 4. Start Frigate

```bash
docker compose up -d
```

### 5. Access Web Interface

Open your browser and navigate to:
```
http://your-server-ip:5000
```

---

## docker-compose.yaml

```yaml
version: "3.9"

services:
  frigate:
    container_name: frigate
    restart: unless-stopped
    image: ghcr.io/blakeblackshear/frigate:stable
    shm_size: "256mb"
    devices:
      # - /dev/bus/usb:/dev/bus/usb  # for Coral USB
      # - /dev/apex_0:/dev/apex_0    # for Coral PCIe
      # - /dev/dri/renderD128        # for Intel GPU
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./config:/config
      - ./storage:/media/frigate
      - type: tmpfs
        target: /tmp/cache
        tmpfs:
          size: 1000000000
    ports:
      - "5000:5000"
      - "8554:8554" # RTSP feeds
      - "8555:8555/tcp" # WebRTC over tcp
      - "8555:8555/udp" # WebRTC over udp
    environment:
      - FRIGATE_RTSP_PASSWORD=password
```

---

## Configuration Examples

### Basic Configuration (config/config.yml)

```yaml
mqtt:
  enabled: False

cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://username:password@camera-ip:554/stream
          roles:
            - detect
            - record
    detect:
      width: 1280
      height: 720
      fps: 5
    record:
      enabled: True
    snapshots:
      enabled: True
```

### Configuration with Multiple Cameras

```yaml
mqtt:
  host: mqtt-broker-ip
  user: mqtt-username
  password: mqtt-password

detectors:
  cpu1:
    type: cpu

cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://username:password@192.168.1.100:554/stream1
          roles:
            - detect
            - record
    detect:
      width: 1920
      height: 1080
      fps: 5
    record:
      enabled: True
      retain:
        days: 7
        mode: motion
    snapshots:
      enabled: True
      retain:
        default: 14
    objects:
      track:
        - person
        - car
        - dog

  backyard:
    ffmpeg:
      inputs:
        - path: rtsp://username:password@192.168.1.101:554/stream1
          roles:
            - detect
            - record
    detect:
      width: 1920
      height: 1080
      fps: 5
    record:
      enabled: True
      retain:
        days: 7
        mode: motion
    snapshots:
      enabled: True
    objects:
      track:
        - person
        - cat
```

### Configuration with Google Coral TPU

```yaml
detectors:
  coral:
    type: edgetpu
    device: usb

mqtt:
  enabled: False

cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://username:password@camera-ip:554/stream
          roles:
            - detect
            - record
    detect:
      width: 1280
      height: 720
      fps: 5
    record:
      enabled: True
    snapshots:
      enabled: True
```

### Configuration with Intel GPU (VAAPI)

```yaml
detectors:
  cpu1:
    type: cpu

ffmpeg:
  hwaccel_args: preset-vaapi

cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://username:password@camera-ip:554/stream
          roles:
            - detect
            - record
    detect:
      width: 1280
      height: 720
      fps: 5
```

---

## Hardware Acceleration

### Google Coral TPU (USB)

Uncomment in docker-compose.yaml:
```yaml
devices:
  - /dev/bus/usb:/dev/bus/usb
```

Update config.yml:
```yaml
detectors:
  coral:
    type: edgetpu
    device: usb
```

### Google Coral TPU (PCIe)

Uncomment in docker-compose.yaml:
```yaml
devices:
  - /dev/apex_0:/dev/apex_0
```

Update config.yml:
```yaml
detectors:
  coral:
    type: edgetpu
    device: pci
```

### Intel GPU (Quick Sync)

Uncomment in docker-compose.yaml:
```yaml
devices:
  - /dev/dri/renderD128
```

Update config.yml:
```yaml
ffmpeg:
  hwaccel_args: preset-vaapi
```

### NVIDIA GPU

Add to docker-compose.yaml:
```yaml
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: 1
          capabilities: [gpu]
```

Update config.yml:
```yaml
ffmpeg:
  hwaccel_args: preset-nvidia-h264
```

---

## Common Camera RTSP URLs

### Amcrest / Dahua
```
rtsp://username:password@ip:554/cam/realmonitor?channel=1&subtype=0
```

### Hikvision
```
rtsp://username:password@ip:554/Streaming/Channels/101
```

### Reolink
```
rtsp://username:password@ip:554/h264Preview_01_main
```

### Wyze (with RTSP firmware)
```
rtsp://username:password@ip:554/live
```

### Generic ONVIF
```
rtsp://username:password@ip:554/stream1
```

---

## Useful Commands

```bash
# Start Frigate
docker compose up -d

# Stop Frigate
docker compose down

# View logs
docker compose logs -f

# Restart Frigate
docker compose restart

# Update Frigate
docker compose pull
docker compose up -d

# Access Frigate shell
docker compose exec frigate bash
```

---

## Home Assistant Integration

Add to Home Assistant's `configuration.yaml`:

```yaml
frigate:
  host: frigate-server-ip:5000
```

Or use the Frigate integration in Home Assistant UI:
1. Go to Settings > Devices & Services
2. Click "+ ADD INTEGRATION"
3. Search for "Frigate"
4. Enter your Frigate URL

---

## Storage Recommendations

### Minimum Storage
- **Recordings**: 50GB per camera for 7 days
- **Snapshots**: 5GB per camera

### Recommended Storage
- **SSD**: For OS and Frigate database
- **HDD**: For recordings and snapshots
- Use `retention` settings to manage storage automatically

---

## Troubleshooting

### High CPU Usage

1. Enable hardware acceleration
2. Reduce detection FPS
3. Lower camera resolution for detect stream
4. Use Google Coral TPU

### Camera Not Detected

1. Verify RTSP URL is correct
2. Check camera credentials
3. Ensure camera is accessible on network
4. Review Frigate logs: `docker compose logs -f`

### Permission Denied on Storage

```bash
# Fix permissions
sudo chown -R 1000:1000 ./storage
sudo chown -R 1000:1000 ./config
```

### Port Already in Use

```bash
# Check what's using the port
sudo lsof -i :5000

# Change port in docker-compose.yaml
ports:
  - "5001:5000"  # Changed from 5000:5000
```

### Low Detection Accuracy

1. Adjust detection threshold in config
2. Use better hardware accelerator (Coral TPU)
3. Improve camera lighting
4. Add motion masks for problematic areas

---

## Performance Tips

1. **Use hardware acceleration** (Coral TPU or GPU)
2. **Adjust FPS**: Lower FPS for detect stream (3-5 fps is usually enough)
3. **Use sub-streams**: Use lower resolution for detection
4. **Enable motion masks**: Exclude areas with constant motion
5. **Optimize retention**: Keep only necessary recordings

---

## Object Detection

Default objects Frigate can detect:
- person
- bicycle
- car
- motorcycle
- airplane
- bus
- train
- truck
- boat
- bird
- cat
- dog
- horse
- sheep
- cow
- elephant
- bear
- zebra
- giraffe

Configure in config.yml:
```yaml
objects:
  track:
    - person
    - car
  filters:
    person:
      min_area: 5000
      max_area: 100000
      threshold: 0.7
```

---

## Resources

- [Frigate Documentation](https://docs.frigate.video/)
- [Frigate GitHub](https://github.com/blakeblackshear/frigate)
- [Home Assistant Integration](https://www.home-assistant.io/integrations/frigate/)
- [Community Forum](https://github.com/blakeblackshear/frigate/discussions)

---

## License

Frigate is licensed under MIT License.

---

**Happy Monitoring! 📹**
