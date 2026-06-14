# Nextcloud Docker Deployment

![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-29.5.3-2496ED?style=flat&logo=docker&logoColor=white)
![Nextcloud](https://img.shields.io/badge/Nextcloud-Latest-0082C9?style=flat&logo=nextcloud&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat&logo=postgresql&logoColor=white)
![Lab Tested](https://img.shields.io/badge/Lab_Tested-Multipass_VM-blueviolet?style=flat)

Self-hosted document management platform — Nextcloud with PostgreSQL backend, deployed via Docker Compose. Includes user/group management, shared folders with permissions, and document versioning.

> Tested in a lab environment on Ubuntu 24.04 LTS via Multipass VM.

---

## Stack

| Component | Version |
|---|---|
| Ubuntu | 24.04 LTS |
| Docker | 29.5.3 |
| Docker Compose | 5.1.4 |
| Nextcloud | latest |
| PostgreSQL | 16 |

---

## Architecture

```
Internet
    ↓
Nginx Reverse Proxy (port 80/443, SSL)
    ↓
Nextcloud (Docker, port 8080)
    ↓
PostgreSQL (Docker, internal)
    ↓
Persistent volumes (app data + database)
```

---

## Requirements

- Ubuntu 24.04 LTS
- 2 vCPU minimum
- 2GB RAM minimum
- 10GB disk minimum
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

### 2. Deploy Nextcloud + PostgreSQL

```bash
sudo mkdir -p /opt/nextcloud
cd /opt/nextcloud
sudo nano docker-compose.yml
```

Use the `docker-compose.yml` from this repository, then:

```bash
sudo docker compose up -d
sudo docker ps
```

### 3. Configure trusted domains

By default, Nextcloud only allows access via `localhost`. Add your domain or IP:

```bash
sudo docker exec -u www-data nextcloud-app-1 php occ config:system:get trusted_domains
sudo docker exec -u www-data nextcloud-app-1 php occ config:system:set trusted_domains 1 --value=your.domain.com
```

### 4. Nginx reverse proxy + SSL

```nginx
server {
    listen 80;
    server_name your.domain.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
sudo certbot --nginx -d your.domain.com
```

---

## User & Group Management

All user/group administration is done via the `occ` command-line tool (interactive flags `-it` required for password prompts).

### Create a group

```bash
sudo docker exec -u www-data nextcloud-app-1 php occ group:add documentation
```

### Create a user and assign to a group

```bash
sudo docker exec -it -u www-data nextcloud-app-1 php occ user:add --group="documentation" jdupont
```

### List users / groups

```bash
sudo docker exec -u www-data nextcloud-app-1 php occ user:list
sudo docker exec -u www-data nextcloud-app-1 php occ group:list
```

---

## Shared Folders & Permissions

Folders can be shared with a group directly from the web interface:

```
Files → New Folder ("Procedures")
Right-click → Share → select group → set read/write permissions
```

This enables document space structuring by confidentiality level (e.g. separate folders/groups for Strategy, Finance, Legal, Technical Documentation).

---

## Versioning

Nextcloud automatically keeps file version history. To view it:

```
Right-click file → Details → Versions tab
```

Previous versions can be restored directly from this panel — no extra configuration required.

---

## Useful Commands

```bash
# Check container status
sudo docker ps

# View logs
sudo docker compose logs -f

# Restart Nextcloud
sudo docker compose restart

# Stop / Start
sudo docker compose down
sudo docker compose up -d
```

---

## File Structure

```
/opt/nextcloud/
├── docker-compose.yml
└── (Docker volumes: db_data, nextcloud_data)

/etc/nginx/sites-available/nextcloud
```

---

## Author

**Adrien Alfred EBOI** — Linux Systems Administrator
- GitHub : [adrienalfred](https://github.com/adrienalfred)

---

## License

MIT
