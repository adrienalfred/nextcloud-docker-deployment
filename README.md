# Self-Hosted Infrastructure — Nextcloud, MinIO & Monitoring

![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-29.5.3-2496ED?style=flat&logo=docker&logoColor=white)
![Nextcloud](https://img.shields.io/badge/Nextcloud-Latest-0082C9?style=flat&logo=nextcloud&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat&logo=postgresql&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-S3_Compatible-C72E49?style=flat&logo=minio&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Monitoring-F46800?style=flat&logo=grafana&logoColor=white)
![Lab Tested](https://img.shields.io/badge/Lab_Tested-Multipass_VM-blueviolet?style=flat)

A complete self-hosted infrastructure stack deployed via Docker Compose: document management (Nextcloud + PostgreSQL), S3-compatible object storage (MinIO), and full system monitoring (Prometheus + Grafana + Node Exporter).

> Tested in a lab environment on Ubuntu 24.04 LTS via Multipass VM.

---

## Architecture

```
Internet
    ↓
Nginx Reverse Proxy (port 80/443, SSL)
    ↓
┌─────────────────────────────────────────────┐
│                                               │
│  Nextcloud (8080) ──► PostgreSQL 16          │
│       │                                      │
│       └──► External Storage (S3) ──► MinIO (9000/9001)
│                                               │
│  Node Exporter (9100) ──► Prometheus (9090) ──► Grafana (3000)
│                                               │
└─────────────────────────────────────────────┘
```

---

## Stack

| Component | Version | Role |
|---|---|---|
| Ubuntu | 24.04 LTS | Host OS |
| Docker / Compose | 29.5.3 / 5.1.4 | Container runtime |
| Nextcloud | latest | Document management (GED) |
| PostgreSQL | 16 | Nextcloud database |
| MinIO | latest | S3-compatible object storage |
| Prometheus | latest | Metrics collection |
| Node Exporter | latest | System metrics exporter |
| Grafana | latest | Monitoring dashboards |

---

## Requirements

- Ubuntu 24.04 LTS
- 2 vCPU minimum
- 2GB RAM minimum
- 15GB disk minimum
- Root or sudo access

---

## Quick Start

### 1. Install Docker

```bash
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 2. Deploy the stack

```bash
sudo mkdir -p /opt/nextcloud
cd /opt/nextcloud
```

Place `docker-compose.yml` and `prometheus.yml` from this repository in this directory, then:

```bash
sudo docker compose up -d
sudo docker ps
```

### 3. Access the services

| Service | URL | Default Credentials |
|---|---|---|
| Nextcloud | `http://<server-ip>:8080` | admin / (set in compose file) |
| MinIO Console | `http://<server-ip>:9001` | minioadmin / (set in compose file) |
| Prometheus | `http://<server-ip>:9090` | — |
| Grafana | `http://<server-ip>:3000` | admin / (set in compose file) |

> ⚠️ Change all default passwords before production use.

---

## 1. Nextcloud — Document Management (GED)

### Configure trusted domains

```bash
sudo docker exec -u www-data nextcloud-app-1 php occ config:system:get trusted_domains
sudo docker exec -u www-data nextcloud-app-1 php occ config:system:set trusted_domains 1 --value=your.domain.com
```

### User & group management

```bash
# Create a group
sudo docker exec -u www-data nextcloud-app-1 php occ group:add documentation

# Create a user and assign to a group
sudo docker exec -it -u www-data nextcloud-app-1 php occ user:add --group="documentation" jdupont

# List users / groups
sudo docker exec -u www-data nextcloud-app-1 php occ user:list
sudo docker exec -u www-data nextcloud-app-1 php occ group:list
```

### Shared folders & permissions

```
Files → New Folder ("Procedures")
Right-click → Share → select group → set read/write permissions
```

Enables document space structuring by confidentiality level (e.g. Strategy, Finance, Legal, Technical Documentation).

### Versioning

```
Right-click file → Details → Versions tab
```

Previous versions can be restored directly — no extra configuration required.

---

## 2. MinIO — S3-Compatible Object Storage

Access the console at `http://<server-ip>:9001` and create buckets to organize content by type:

```
client-documents/
contracts/
invoices/
applications/
```

This storage layer is designed to be consumed by future front-office applications via the S3 API, and can also be connected to Nextcloud as **External Storage** (Settings → Administration → External Storage → Amazon S3).

---

## 3. Monitoring — Prometheus + Grafana + Node Exporter

### How it works

```
Node Exporter exposes host metrics (CPU, RAM, disk, network)
        ↓
Prometheus scrapes Node Exporter every 15s (see prometheus.yml)
        ↓
Grafana queries Prometheus and renders dashboards
```

### Setup

1. In Grafana, add Prometheus as a data source: `http://prometheus:9090`
2. Import the **Node Exporter Full** dashboard (ID `1860` from grafana.com)
3. Select the Prometheus data source when prompted

This provides real-time visibility on CPU usage, memory, disk space, network traffic, and system uptime — the foundation for proactive infrastructure monitoring.

---

## Useful Commands

```bash
# Check all containers
sudo docker ps

# View logs for a specific service
sudo docker compose logs -f <service-name>

# Restart everything
sudo docker compose restart

# Stop / Start the stack
sudo docker compose down
sudo docker compose up -d
```

---

## File Structure

```
/opt/nextcloud/
├── docker-compose.yml
├── prometheus.yml
└── (Docker volumes: db_data, nextcloud_data, minio_data,
     prometheus_data, grafana_data)

/etc/nginx/sites-available/nextcloud
```

---

## Author

**Adrien Alfred EBOI** — Linux Systems Administrator
- GitHub : [adrienalfred](https://github.com/adrienalfred)

---

## License

MIT
