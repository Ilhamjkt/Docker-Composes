# Docker Installation Guide

⚠️ **Warning**: This guide assumes you're using **root** privileges. Use at your own risk.

---

## Quick Start

Install Docker Engine and Docker Compose plugin with a single command:

```bash
curl -sSL get.docker.com | sh
```

---

## Detailed Installation Instructions

### Debian / Ubuntu

#### 1. Update and Install Dependencies

```bash
apt update
apt install ca-certificates curl
```

#### 2. Add Docker GPG Key

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

#### 3. Add Docker Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
```

#### 4. Install Docker

```bash
apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

---

### RHEL / Alma / Rocky / CentOS

#### 1. Install Required Tools

```bash
yum install -y yum-utils
```

#### 2. Add Docker Repository

```bash
yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

#### 3. Install Docker

```bash
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 4. Start and Enable Docker

```bash
systemctl enable docker && systemctl start docker
```

---

### Fedora

#### 1. Install Required Tools

```bash
dnf -y install dnf-plugins-core
```

#### 2. Add Docker Repository

```bash
dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

#### 3. Install Docker

```bash
dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 4. Start and Enable Docker

```bash
systemctl enable docker && systemctl start docker
```

---

## Verification

After installation, verify Docker is working properly:

```bash
docker --version
docker compose version
```

---

## Basic Docker Commands

```bash
# Run a container
docker run hello-world

# List running containers
docker ps

# List all containers
docker ps -a

# Stop a container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# List images
docker images

# Remove an image
docker rmi <image_id>
```

---

## Basic Docker Compose Commands

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# Restart services
docker compose restart

# Rebuild and start
docker compose up -d --build
```

---

## docker-compose.yaml Template

```yaml
services:
  app:
    image: nginx:alpine
    container_name: my-app
    ports:
      - "80:80"
    volumes:
      - ./data:/usr/share/nginx/html
    restart: unless-stopped
```

---

## Troubleshooting

### Permission Denied

```bash
# Add user to docker group
sudo usermod -aG docker $USER
# Logout and login again
```

### Check Docker Status

```bash
systemctl status docker
```

### View Docker Logs

```bash
journalctl -u docker.service
```

---

> **Tip**: For non-root usage, add your user to the docker group: `sudo usermod -aG docker $USER` then logout and login again.

---

## References

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

**Happy Dockerizing! 🐳**
