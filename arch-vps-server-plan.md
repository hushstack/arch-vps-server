# Arch VPS Server Lab Plan

This plan is for creating a **local VPS-style server environment** on an Arch Linux laptop.

Main path:

```bash
/mnt/storage/arch-vps-server
```

Goal:

- Use the laptop like a local VPS lab.
- Run backend projects with Docker.
- Clone backend projects into one organized folder.
- Use a reverse proxy like a real VPS.
- Configure local domains.
- Optionally expose real domains to the internet with Cloudflare Tunnel.

---

## 1. Big Picture

Your Arch laptop will act like this:

```text
Arch laptop
└── /mnt/storage/arch-vps-server
    ├── Docker
    ├── Caddy reverse proxy
    ├── backend projects
    ├── databases
    ├── logs
    ├── backups
    └── optional Cloudflare Tunnel
```

Real VPS style:

```text
Internet / Browser
        ↓
Domain or local domain
        ↓
Caddy reverse proxy
        ↓
Docker backend container
        ↓
Database container
```

Important:

```text
Laptop ON + Docker running + Proxy running = website online
Laptop OFF / sleep / no internet = website offline
```

---

## 2. Recommended Folder Structure

Create this structure:

```text
/mnt/storage/arch-vps-server/
├── proxy/
│   ├── docker-compose.yml
│   └── Caddyfile
├── projects/
│   ├── backend-one/
│   ├── backend-two/
│   └── website-one/
├── databases/
│   ├── postgres/
│   └── redis/
├── tunnels/
│   └── cloudflare/
├── scripts/
│   ├── start-proxy.sh
│   ├── stop-proxy.sh
│   ├── restart-proxy.sh
│   ├── list-containers.sh
│   └── update-project.sh
├── backups/
├── logs/
└── README.md
```

Purpose:

| Folder | Purpose |
|---|---|
| `proxy/` | Caddy reverse proxy config |
| `projects/` | Backend/frontend projects cloned from Git |
| `databases/` | Persistent database volumes or configs |
| `tunnels/` | Cloudflare Tunnel config |
| `scripts/` | Helper scripts |
| `backups/` | Manual backups |
| `logs/` | Logs and notes |

---

## 3. Install Docker on Arch

Install Docker:

```bash
sudo pacman -S docker docker-compose
```

Enable Docker:

```bash
sudo systemctl enable --now docker
```

Allow your user to run Docker:

```bash
sudo usermod -aG docker "$USER"
```

Then log out and log back in.

Test Docker:

```bash
docker run hello-world
```

---

## 4. Create Main Directory

```bash
sudo mkdir -p /mnt/storage/arch-vps-server/{proxy,projects,databases,tunnels/cloudflare,scripts,backups,logs}
sudo chown -R "$USER:$USER" /mnt/storage/arch-vps-server
cd /mnt/storage/arch-vps-server
```

---

## 5. Create Shared Docker Network

Create one shared network for all projects:

```bash
docker network create arch-vps-net
```

All apps and proxy should join this network.

Why?

Because Caddy can reverse proxy by container name:

```text
api-one:3000
admin-panel:4000
website-one:8080
```

---

## 6. Reverse Proxy with Caddy

Caddy will receive domain traffic and forward it to the correct container.

Example:

```text
api.localhost.test    -> api container
app.localhost.test    -> frontend container
admin.localhost.test  -> admin container
```

### `/mnt/storage/arch-vps-server/proxy/docker-compose.yml`

```yaml
services:
  caddy:
    image: caddy:2-alpine
    container_name: arch-vps-caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - arch-vps-net

volumes:
  caddy_data:
  caddy_config:

networks:
  arch-vps-net:
    external: true
```

### `/mnt/storage/arch-vps-server/proxy/Caddyfile`

Start simple:

```caddyfile
:80 {
    respond "Arch VPS Server Lab is running"
}
```

Run it:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose up -d
```

Test:

```bash
curl http://localhost
```

Expected result:

```text
Arch VPS Server Lab is running
```

---

## 7. Local Domain Setup

Edit hosts file:

```bash
sudo nano /etc/hosts
```

Add local domains:

```text
127.0.0.1 archvps.local
127.0.0.1 api.localhost.test
127.0.0.1 app.localhost.test
127.0.0.1 admin.localhost.test
```

Then update `Caddyfile`:

```caddyfile
archvps.local {
    respond "Arch VPS Server Lab is running"
}
```

Restart Caddy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

Open:

```text
http://archvps.local
```

---

## 8. Example Backend Project Setup

Example project path:

```bash
/mnt/storage/arch-vps-server/projects/my-backend
```

Backend `docker-compose.yml` example:

```yaml
services:
  my-backend:
    build: .
    container_name: my-backend
    restart: unless-stopped
    expose:
      - "3000"
    env_file:
      - .env
    networks:
      - arch-vps-net

networks:
  arch-vps-net:
    external: true
```

Important:

Use `expose`, not always `ports`, when using Caddy.

Why?

- `expose: "3000"` makes the service reachable inside Docker network.
- Caddy can reach it.
- It does not expose every project directly to your host.

Then Caddy can route to it:

```caddyfile
api.localhost.test {
    reverse_proxy my-backend:3000
}
```

Run backend:

```bash
cd /mnt/storage/arch-vps-server/projects/my-backend
docker compose up -d --build
```

Restart proxy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

Open:

```text
http://api.localhost.test
```

---

## 9. Public Domain Option

For public access, use Cloudflare Tunnel.

Public architecture:

```text
yourdomain.com
        ↓
Cloudflare DNS
        ↓
Cloudflare Tunnel
        ↓
your Arch laptop
        ↓
Caddy reverse proxy
        ↓
Docker containers
```

Good for:

- No router port forwarding.
- Works behind home Wi-Fi.
- You can use real domains/subdomains.
- Safer than exposing your laptop directly.

Example public domain mapping:

```text
api.yourdomain.com    -> localhost:80 -> Caddy -> my-backend:3000
app.yourdomain.com    -> localhost:80 -> Caddy -> frontend:8080
```

Important:

The public website is online only when:

```text
Laptop is on
Docker is running
Caddy is running
Cloudflare Tunnel is running
Internet is connected
```

---

## 10. Useful Commands

List containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

View proxy logs:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose logs -f
```

Restart proxy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

Start a project:

```bash
cd /mnt/storage/arch-vps-server/projects/my-project
docker compose up -d --build
```

Stop a project:

```bash
cd /mnt/storage/arch-vps-server/projects/my-project
docker compose down
```

Update a project:

```bash
cd /mnt/storage/arch-vps-server/projects/my-project
git pull
docker compose up -d --build
```

---

## 11. Security Notes

Do not expose databases directly to the internet.

Bad:

```yaml
ports:
  - "5432:5432"
```

Better:

```yaml
expose:
  - "5432"
```

Use `.env` files for secrets.

Do not commit `.env` to Git.

Add this to `.gitignore`:

```gitignore
.env
*.log
data/
```

For public hosting:

- Use HTTPS.
- Keep Docker images updated.
- Do not run random containers from untrusted sources.
- Use strong passwords.
- Back up databases.
- Do not expose your personal laptop ports unless you understand the risk.

---

## 12. Recommended Workflow

For every backend project:

```bash
cd /mnt/storage/arch-vps-server/projects
git clone <project-url>
cd <project-folder>
docker compose up -d --build
```

Then add domain in Caddy:

```caddyfile
project.localhost.test {
    reverse_proxy project-container-name:project-port
}
```

Then restart proxy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

---

## 13. Future Upgrade Path

Local laptop lab:

```text
Arch laptop + Docker + Caddy + local domains
```

Later real VPS:

```text
Ubuntu/Debian VPS + Docker + Caddy + real domain
```

The workflow will be almost the same:

```bash
git clone project
docker compose up -d --build
```

That is why this local setup is good practice.
