# Raspberry Pi Home Server

Self-hosted Raspberry Pi home server stack using Docker, Nginx, Samba, Grafana, Prometheus, Uptime Kuma, Jellyfin and Pi-hole.

This project documents a Linux-based home server running on a Raspberry Pi.

## Overview

The server provides:

- Media streaming with Jellyfin
- Uptime monitoring with Uptime Kuma
- System monitoring with Prometheus/Grafana
- Network-wide DNS filtering with Pi-hole
- SMB file sharing with Samba
- Hostname-based routing with Nginx
- Firewall and security with UFW and Fail2Ban


## Stack

- Docker
- Docker Compose
- Nginx
- Samba
- Jellyfin
- Uptime Kuma
- Grafana
- Prometheus
- Node Exporter
- Pi-hole
- UFW
- Fail2Ban


## Services

| Service | Purpose | Port |
|---|---|---|
| Jellyfin | Media server | 8096 |
| Uptime Kuma | Uptime monitoring | 3001 |
| Grafana | Monitoring dashboards | 3000 |
| Prometheus | Metrics collection | 9090 |
| Node Exporter | Host system metrics | 9100 |
| Pi-hole | DNS/ad blocking | 53, 8081 |
| Nginx | Reverse proxy | 80 |
| Samba | Network shares | 445 |


## Architecture

```text
Client devices
     |
     v
Nginx reverse proxy
     |
     +--> Jellyfin
     +--> Grafana
     +--> Prometheus
     +--> Uptime Kuma

Prometheus --> Node Exporter
Grafana ----> Prometheus
Pi-hole ----> Local DNS / ad blocking
Samba ------> Shared folders
```


## Local Hostnames

| Hostname | Service |
|---|---|
| jellyfin.home | Jellyfin |
| grafana.home | Grafana |
| kuma.home | Uptime Kuma |
| prometheus.home | Prometheus |

These hostnames require local DNS records, such as Pi-hole local DNS entries, or manual hosts file entries on client devices.

The services can also be accessed directly by IP address:

| URL | Service |
|---|---|
| `http://<server-ip>:8096` | Jellyfin |
| `http://<server-ip>:3001` | Uptime Kuma |
| `http://<server-ip>:3000` | Grafana |
| `http://<server-ip>:9090` | Prometheus |
| `http://<server-ip>:8081/admin` | Pi-hole |

## Repository Structure

```text
.
├── README.md
├── docker-compose.yml
├── .env.example
├── .gitignore
├── nginx
│   └── homeserver.conf
├── prometheus
│   └── prometheus.yml
├── samba
│   └── smb.conf.example
└── scripts
    ├── install_docker.sh
    └── update_containers.sh
```

## Setup

### 1. Clone the repository

```bash
git clone <repo-url>
cd Linux-home-server
```

### 2. Create the environment file

Docker Compose uses `.env`, not `.env.example`.

Copy the example file:

```bash
cp .env.example .env
```

Edit the values:

```bash
nano .env
```

Example:

```env
HOST_IP=192.168.0.50
TZ=Australia/Melbourne
PIHOLE_PASSWORD=change-me
```

Do not commit the real `.env` file.

### 3. Install Docker

Run:

```bash
./scripts/install-docker.sh
```

After installation, log out and back in so Docker group permissions apply.

Alternatively, for the current session only:

```bash
newgrp docker
```

Check Docker:

```bash
docker --version
docker compose version
```

### 4. Start the stack

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

Validate the Compose file without starting containers:

```bash
docker compose config
```

## Updating Containers

Run:

```bash
./scripts/update-containers.sh
```

This pulls newer container images and removes unused services.

## Nginx Reverse Proxy

The Nginx config is at:

```text
nginx/homeserver.conf
```

Copy it into Nginx:

```bash
sudo cp nginx/homeserver.conf /etc/nginx/sites-available/homeserver
sudo ln -s /etc/nginx/sites-available/homeserver /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

If `nginx -t` fails, fix the config before restarting Nginx.

## Samba File Shares

The example Samba config is stored at:

```text
samba/smb.conf.example
```

Copy or merge the example into:

```text
/etc/samba/smb.conf
```

Create the shared folders:

```bash
mkdir -p ~/shared ~/media
```

Add a Samba password for the current Linux user:

```bash
sudo smbpasswd -a $USER
```

Restart Samba:

```bash
sudo systemctl restart smbd
```

Check Samba status:

```bash
sudo systemctl status smbd
```


## Prometheus

Prometheus is configured using:

```text
prometheus/prometheus.yml
```

By default, it scrapes Node Exporter on:

```text
192.168.0.50:9100
```

Update this target if your server IP address is different.



## Firewall

Example UFW rules:

```bash
sudo ufw allow ssh
sudo ufw allow samba
sudo ufw allow 80/tcp
sudo ufw allow 3001/tcp
sudo ufw allow 8096/tcp
sudo ufw allow 7359/udp
sudo ufw allow 3000/tcp
sudo ufw allow 9090/tcp
sudo ufw allow 9100/tcp
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw allow 8081/tcp
sudo ufw enable
```

Check firewall status:

```bash
sudo ufw status
```


## Common Issues

### `.env` file missing

If `.env` is missing, Compose may not load values such as `TZ` and `PIHOLE_PASSWORD`.

Fix:

```bash
cp .env.example .env
```

### Pi-hole port 53 conflict

Pi-hole requires port 53 for DNS. Some Ubuntu systems, port 53 may already be used by `systemd-resolved`.

Check with:

```bash
sudo ss -tulpn | grep ':53'
```

### Existing containers cause name conflicts

If containers already exist from manual `docker run` commands, remove them first:

```bash
docker rm -f jellyfin grafana prometheus node-exporter uptime-kuma pihole
```

Then restart:

```bash
docker compose up -d
```


### Docker permission denied

If Docker was just installed, try logging out and back in, or run:

```bash
newgrp docker
```


