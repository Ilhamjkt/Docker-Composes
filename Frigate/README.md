# Frigate NVR – Docker Compose Deployment

A structured and self-contained Docker Compose setup for deploying **Frigate NVR** in a self-hosted environment.

This repository focuses on clean deployment, maintainable configuration, and production-ready structure — without unnecessary complexity.

---

## 📦 Overview

Frigate is a local Network Video Recorder (NVR) designed for real-time object detection using IP cameras.

This repository provides:
- Organized Docker Compose configuration
- Structured config directory
- Easy customization for home or lab environments
- Clean deployment workflow

---

## 📁 Project Structure

```
Frigate/
├── Docker-compose.yml
├── config/
│   └── config.yml
└── README.md
```

---

## 🚀 Requirements

Before deployment, ensure the following are installed:

- Docker (latest stable recommended)
- Docker Compose
- Linux host system (recommended for stability)
- IP Camera with RTSP stream

Optional (recommended for performance):
- NVIDIA GPU
- Google Coral TPU
- Other supported hardware accelerators

---

## ⚙️ Configuration

### 1️⃣ Edit Docker Compose

Open:

```
Docker-compose.yml
```

Adjust:
- Port mappings
- Storage paths
- Environment variables
- Shared memory size (shm_size)

If running multiple cameras, increase shared memory accordingly.

---

### 2️⃣ Configure Cameras

Open:

```
config/config.yml
```

Define:
- RTSP stream URLs
- Detection resolution
- Frames per second
- Recording behavior

Make sure to:
- Replace placeholder usernames/passwords
- Verify camera stream accessibility

---

## ▶️ Deploy Frigate

From inside the `Frigate` directory:

```bash
docker compose up -d
```

To monitor logs:

```bash
docker compose logs -f
```

To stop the service:

```bash
docker compose down
```

---

## 🌐 Access Web Interface

After deployment, open:

```
http://YOUR_SERVER_IP:PORT
```

Replace:
- `YOUR_SERVER_IP` with your server address
- `PORT` with the port defined in Docker Compose

---

## 📊 Performance Notes

- Increase `shm_size` if you experience crashes.
- Use hardware acceleration for better detection speed.
- Store recordings on fast storage (SSD recommended).
- Keep your camera FPS reasonable (5–10 FPS for detection is usually sufficient).

---

## 🔐 Environment Variables (Optional)

For better security and maintainability, consider using a `.env` file:

Example:

```
FRIGATE_RTSP_PASSWORD=your_password_here
```

Then reference it inside `Docker-compose.yml`.

---

## 🛠 Troubleshooting

If the container fails to start:

1. Check logs:
   ```bash
   docker compose logs
   ```
2. Validate YAML indentation
3. Confirm storage paths exist
4. Ensure ports are not already in use

---

## 🔄 Updating

To update Frigate:

```bash
docker compose pull
docker compose up -d
```

---

## 🤝 Contributions

Contributions, improvements, and suggestions are welcome.

If you fork this repository:
1. Create a new branch
2. Make changes
3. Submit a Pull Request

---

## 📄 License

Add your preferred license (e.g., MIT License) to define usage rights and redistribution rules.

---

## ⚠️ Disclaimer

This repository provides deployment configuration only.  
Ensure compliance with local regulations regarding camera recording and data retention.

---

**Maintained by Ilham**
