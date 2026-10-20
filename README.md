# Docker Installation Guide

Install **Docker Engine** and the **Docker Compose plugin** (`docker compose`) from Docker's official repositories, then use the compose files in this repo.

⚠️ **Warning**: Commands below assume you are **root**. If you are not, prefix them with `sudo` (or run `sudo -i` first).

---

## Quick Start

Docker's convenience script detects your distro and installs Docker Engine + Compose plugin:

```bash
curl -fsSL https://get.docker.com | sh
systemctl enable --now docker
```

> Docker recommends the script for test/dev machines. For servers, use the per-distro steps below so you control the repository and versions.

---

## Detailed Installation Instructions

Supported: Debian, Ubuntu (and derivatives like Mint/Pop!_OS), RHEL / Rocky / Alma, CentOS Stream, Fedora.

### Debian

#### 1. Remove conflicting packages

```bash
apt remove $(dpkg --get-selections docker.io docker-compose docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```

`apt` may report that none of these are installed — that's fine.

#### 2. Add Docker GPG key

```bash
apt update
apt install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

#### 3. Add Docker repository

```bash
tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
apt update
```

#### 4. Install Docker

```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now docker
```

---

### Ubuntu

Same as Debian, but with the **Ubuntu** repository (do not use the Debian repo on Ubuntu).

#### 1. Remove conflicting packages

```bash
apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```

#### 2. Add Docker GPG key

```bash
apt update
apt install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

#### 3. Add Docker repository

```bash
tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
apt update
```

`UBUNTU_CODENAME` makes this work on Ubuntu derivatives (Linux Mint, Pop!_OS) too.

#### 4. Install Docker

```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now docker
```

---

### RHEL / Rocky / Alma

#### 1. Remove conflicting packages

```bash
dnf remove docker docker-client docker-client-latest docker-common docker-latest \
  docker-latest-logrotate docker-logrotate docker-engine podman runc
```

#### 2. Add Docker repository

```bash
dnf -y install dnf-plugins-core
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

#### 3. Install Docker

```bash
dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Accept the GPG key if prompted; its fingerprint must be `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`.

#### 4. Start and enable Docker

```bash
systemctl enable --now docker
```

---

### CentOS Stream

Same as RHEL, but with the **CentOS** repository:

```bash
dnf remove docker docker-client docker-client-latest docker-common docker-latest \
  docker-latest-logrotate docker-logrotate docker-engine
dnf -y install dnf-plugins-core
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl enable --now docker
```

---

### Fedora

Fedora 41+ ships **dnf5**, whose `config-manager` syntax differs from older releases.

#### 1. Remove conflicting packages

```bash
dnf remove docker docker-client docker-client-latest docker-common docker-latest \
  docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
```

#### 2. Add Docker repository

```bash
dnf -y install dnf-plugins-core
dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo
```

On Fedora 40 and older (dnf4) use: `dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo`

#### 3. Install Docker

```bash
dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 4. Start and enable Docker

```bash
systemctl enable --now docker
```

---

## Verification

```bash
docker --version
docker compose version
docker run --rm hello-world
```

`hello-world` printing "Hello from Docker!" means the engine can pull and run containers.

> **Docker Compose v1 vs v2**: this guide installs Compose **v2** as a Docker plugin, used as `docker compose` (space). The old standalone `docker-compose` (hyphen, v1) is deprecated — don't install it, and use `docker compose` in every command.

---

## Running Docker as a non-root user

```bash
usermod -aG docker <username>
# log out and back in (or run: newgrp docker)
docker run --rm hello-world
```

⚠️ Members of the `docker` group effectively have **root access** to the host. Only add users you trust.

---

## Updating Docker

```bash
# Debian / Ubuntu
apt update && apt install --only-upgrade -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# RHEL / Rocky / Alma / CentOS Stream / Fedora
dnf upgrade -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Running containers with `restart: unless-stopped` start again automatically after the engine restarts.

---

## Uninstalling Docker

```bash
# Debian / Ubuntu
apt purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
rm /etc/apt/sources.list.d/docker.sources /etc/apt/keyrings/docker.asc

# RHEL / Rocky / Alma / CentOS Stream / Fedora
dnf remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

# Delete ALL images, containers, and volumes (irreversible)
rm -rf /var/lib/docker /var/lib/containerd
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

# Pull newer images and recreate containers
docker compose pull && docker compose up -d
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

### Permission denied on `/var/run/docker.sock`

Your user is not in the `docker` group — see [Running Docker as a non-root user](#running-docker-as-a-non-root-user).

### `docker: 'compose' is not a docker command`

The Compose plugin is missing. Install `docker-compose-plugin` with your package manager (step 4 of your distro).

### Check Docker status

```bash
systemctl status docker
```

### View Docker logs

```bash
journalctl -u docker.service
```

---

## References

- [Install Docker Engine](https://docs.docker.com/engine/install/)
- [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

**Happy Dockerizing! 🐳**
