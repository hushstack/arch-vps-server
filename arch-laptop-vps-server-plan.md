# Arch Laptop VPS Server Plan

This plan is for using an **Arch Linux laptop as a VPS-style server**.

Main path:

```bash
/mnt/storage/arch-vps-server
```

## Goal

- Use my Arch laptop like a VPS server.
- Run backend/frontend projects with Docker.
- Use Caddy as a reverse proxy.
- Support local domains for testing.
- Support real public domains using DNS A records.
- Support optional Cloudflare Tunnel if A record / port forwarding does not work.
- Make it easy to move the same setup to a real VPS later.

---

## 1. Big Picture

### Local VPS-style server

```text
Arch laptop
└── /mnt/storage/arch-vps-server
    ├── Docker
    ├── Caddy reverse proxy
    ├── backend projects
    ├── frontend projects
    ├── databases
    ├── logs
    ├── backups
    └── optional Cloudflare Tunnel
```

### Public domain flow with DNS A record

```text
Other user
    ↓
https://api.yourdomain.com
    ↓
Cloudflare DNS A record
    ↓
Home public IPv4 address
    ↓
Router port forwarding
    ↓
Arch laptop
    ↓
Caddy reverse proxy
    ↓
Docker backend container
```

### Optional public domain flow with Cloudflare Tunnel

```text
Other user
    ↓
https://api.yourdomain.com
    ↓
Cloudflare
    ↓
Cloudflare Tunnel
    ↓
Arch laptop
    ↓
Caddy reverse proxy
    ↓
Docker backend container
```

---

## 2. Important Rule

This setup works publicly only when:

```text
Laptop is ON
Laptop does not sleep
Internet is connected
Docker is running
Caddy is running
Backend containers are running
Domain DNS is configured correctly
```

If using DNS A record, this is also required:

```text
Home network has public IPv4
Router forwards ports 80 and 443 to laptop
ISP does not block ports 80 and 443
ISP is not using CGNAT
```

If DNS A record does not work because of CGNAT or blocked ports, use **Cloudflare Tunnel** instead.

---

## 3. Recommended Folder Structure

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
│   ├── create-network.sh
│   ├── start-proxy.sh
│   ├── stop-proxy.sh
│   ├── restart-proxy.sh
│   ├── list-containers.sh
│   ├── check-public-ip.sh
│   └── update-project.sh
├── backups/
├── logs/
├── .gitignore
└── README.md
```

### Folder Purpose

| Folder | Purpose |
|---|---|
| `proxy/` | Caddy reverse proxy config |
| `projects/` | Backend/frontend projects cloned from Git |
| `databases/` | Database data/config folders |
| `tunnels/` | Cloudflare Tunnel config |
| `scripts/` | Helper scripts |
| `backups/` | Manual backups |
| `logs/` | Log files and notes |

---

## 4. Create Main Directory

```bash
sudo mkdir -p /mnt/storage/arch-vps-server/{proxy,projects,databases/postgres,databases/redis,tunnels/cloudflare,scripts,backups,logs}
sudo chown -R "$USER:$USER" /mnt/storage/arch-vps-server
cd /mnt/storage/arch-vps-server
```

---

## 5. Install Docker on Arch

Install Docker:

```bash
sudo pacman -S docker docker-compose
```

Enable Docker:

```bash
sudo systemctl enable --now docker
```

Allow current user to run Docker:

```bash
sudo usermod -aG docker "$USER"
```

Then log out and log back in.

Test Docker:

```bash
docker run hello-world
```

---

## 6. Create Shared Docker Network

Create one shared Docker network:

```bash
docker network create arch-vps-net
```

All projects and Caddy should use this network.

Why?

Because Caddy can reach containers by container name:

```text
my-backend:3000
my-frontend:8080
postgres:5432
redis:6379
```

---

## 7. Caddy Reverse Proxy Setup

Caddy receives traffic on ports `80` and `443`, then forwards traffic to the correct Docker container.

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

Start with this:

```caddyfile
:80 {
    respond "Arch VPS Server Lab is running"
}
```

Start proxy:

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

## 8. Local Domain Setup

Local domains only work on your laptop unless you also edit DNS/router.

Edit hosts file:

```bash
sudo nano /etc/hosts
```

Add:

```text
127.0.0.1 archvps.local
127.0.0.1 api.localhost.test
127.0.0.1 app.localhost.test
127.0.0.1 admin.localhost.test
```

Update Caddyfile:

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

## 9. DNS A Record Public Domain Mode

This mode makes the laptop behave like a real VPS using a real domain.

### Requirements

```text
Laptop is ON
Docker is running
Caddy is running
Backend container is running
Laptop has static LAN IP
Router forwards ports 80 and 443 to laptop
Cloudflare DNS A record points to home public IPv4
ISP does not block ports 80/443
ISP is not using CGNAT
```

### Traffic flow

```text
Other user
    ↓
https://api.yourdomain.com
    ↓
Cloudflare DNS A record
    ↓
Home public IPv4
    ↓
Router port forwarding
    ↓
Arch laptop
    ↓
Caddy
    ↓
Docker backend
```

### Step 1: Check public IP

On laptop:

```bash
curl ifconfig.me
```

Then check router admin page and find:

```text
WAN IP
Internet IP
Public IP
```

Compare:

```text
curl ifconfig.me IP == router WAN IP
```

If same, A record can probably work.

If different, ISP may use CGNAT. A record may not work. Use Cloudflare Tunnel instead.

### Step 2: Set static LAN IP for laptop

In router settings, create DHCP reservation:

```text
Laptop MAC address -> 192.168.1.50
```

Example laptop LAN IP:

```text
192.168.1.50
```

### Step 3: Router port forwarding

Forward public ports to laptop:

```text
Public port 80  -> 192.168.1.50:80
Public port 443 -> 192.168.1.50:443
```

### Step 4: Cloudflare DNS A records

In Cloudflare DNS:

```text
Type: A
Name: api
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only
```

For root domain:

```text
Type: A
Name: @
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only
```

For app:

```text
Type: A
Name: app
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only
```

Example:

```text
api.yourdomain.com -> 123.45.67.89
app.yourdomain.com -> 123.45.67.89
yourdomain.com     -> 123.45.67.89
```

### Step 5: Caddy real domain config

Example:

```caddyfile
api.yourdomain.com {
    reverse_proxy my-backend:3000
}

app.yourdomain.com {
    reverse_proxy my-frontend:8080
}

yourdomain.com {
    reverse_proxy my-website:8080
}
```

Restart Caddy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

Now other users can access:

```text
https://api.yourdomain.com
https://app.yourdomain.com
https://yourdomain.com
```

---

## 10. Cloudflare Tunnel Public Domain Mode

Use this if DNS A record does not work.

Common reasons A record does not work:

```text
ISP uses CGNAT
ISP blocks port 80/443
Router does not support port forwarding
Home IP changes often
You do not want to expose your router ports
```

Cloudflare Tunnel flow:

```text
Other user
    ↓
https://api.yourdomain.com
    ↓
Cloudflare
    ↓
Cloudflare Tunnel
    ↓
localhost:80
    ↓
Caddy
    ↓
Docker backend
```

Example mapping:

```text
api.yourdomain.com -> http://localhost:80
app.yourdomain.com -> http://localhost:80
```

Caddy still decides where to send traffic:

```caddyfile
api.yourdomain.com {
    reverse_proxy my-backend:3000
}

app.yourdomain.com {
    reverse_proxy my-frontend:8080
}
```

---

## 11. Example Backend Project Setup

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
- It does not expose every project directly to the host/public internet.

Run backend:

```bash
cd /mnt/storage/arch-vps-server/projects/my-backend
docker compose up -d --build
```

Then Caddy can route to it:

```caddyfile
api.yourdomain.com {
    reverse_proxy my-backend:3000
}
```

Restart proxy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

Open:

```text
https://api.yourdomain.com
```

---

## 12. Useful Commands

List running containers:

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

## 13. Security Notes

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
*.env
*.log
logs/
backups/
data/
node_modules/
dist/
build/
```

For public hosting:

- Use HTTPS.
- Keep Docker images updated.
- Do not run random containers from untrusted sources.
- Use strong passwords.
- Back up databases.
- Do not expose database ports publicly.
- Do not expose your personal laptop ports unless you understand the risk.
- Keep Arch updated.
- Keep router firmware updated if possible.

---

## 14. Recommended Workflow

For every backend project:

```bash
cd /mnt/storage/arch-vps-server/projects
git clone <project-url>
cd <project-folder>
docker compose up -d --build
```

Then add domain in Caddy:

```caddyfile
project.yourdomain.com {
    reverse_proxy project-container-name:project-port
}
```

Then restart proxy:

```bash
cd /mnt/storage/arch-vps-server/proxy
docker compose restart
```

---

## 15. Future Upgrade Path

Current laptop VPS-style server:

```text
Arch laptop + Docker + Caddy + domain A record or Cloudflare Tunnel
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

That is why this laptop setup is good practice.

---

# Codex CLI Prompt

Use this prompt when opening Codex CLI inside:

```bash
cd /mnt/storage/arch-vps-server
codex
```

Paste this:

```text
I want you to set up this folder as my local Arch laptop VPS-style server.

Current working directory:
/mnt/storage/arch-vps-server

Goal:
Create a clean Docker-based local VPS environment where I can clone backend/frontend projects, run them with Docker Compose, route domains through Caddy, and later expose real domains using DNS A record or Cloudflare Tunnel.

Please create and configure this structure:

/mnt/storage/arch-vps-server/
├── proxy/
│   ├── docker-compose.yml
│   └── Caddyfile
├── projects/
├── databases/
│   ├── postgres/
│   └── redis/
├── tunnels/
│   └── cloudflare/
├── scripts/
│   ├── create-network.sh
│   ├── start-proxy.sh
│   ├── stop-proxy.sh
│   ├── restart-proxy.sh
│   ├── list-containers.sh
│   ├── check-public-ip.sh
│   └── update-project.sh
├── backups/
├── logs/
├── .gitignore
└── README.md

Requirements:
1. Use Docker Compose.
2. Use Caddy as the reverse proxy.
3. Use one shared external Docker network named arch-vps-net.
4. Caddy should expose ports 80 and 443.
5. Initial Caddyfile should respond with: "Arch VPS Server Lab is running".
6. Scripts should be safe, readable, and executable.
7. Scripts should not destroy containers or volumes unless clearly intended.
8. README.md should explain how to:
   - install Docker on Arch
   - create the Docker network
   - start the proxy
   - stop the proxy
   - add a backend project
   - add a local domain
   - configure DNS A record with Cloudflare
   - configure router port forwarding
   - check public IP and CGNAT
   - use Cloudflare Tunnel later
9. Add an example backend docker-compose.yml snippet in the README.
10. Add an example Caddy reverse_proxy config in the README.
11. Add a useful .gitignore for env files, logs, backups, and local data.
12. Keep the project clean, simple, and easy to maintain.

Important:
- Do not assume this is a cloud VPS.
- This is a local VPS-style server running on my Arch laptop.
- Do not install system packages automatically unless I explicitly ask.
- Only create files and folders inside the current directory.
- Before making changes, inspect the current directory.
- After creating files, show me the final tree and explain how to run it.

Expected commands after setup should be:

docker network create arch-vps-net
cd proxy
docker compose up -d

Then I should be able to test:

curl http://localhost

Expected output:

Arch VPS Server Lab is running
```
