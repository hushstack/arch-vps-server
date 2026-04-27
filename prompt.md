I want you to set up this folder as my local Arch laptop VPS-style server.

Current working directory:
/mnt/storage/arch-vps-server

Goal:
Create a clean Docker-based VPS-style environment on my Arch laptop where I can clone backend/frontend projects, run them with Docker Compose, route domains through Caddy, and support real public domains using DNS A records like a real VPS.

This setup should support two public access modes:

1. DNS A Record Mode:
   Real domain -> Cloudflare DNS A record -> my home public IPv4 -> router port forwarding -> my Arch laptop -> Caddy -> Docker container

2. Cloudflare Tunnel Mode:
   Real domain -> Cloudflare Tunnel -> my Arch laptop -> Caddy -> Docker container

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
│   ├── add-local-domain-example.sh
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
7. Scripts should not destroy containers, images, networks, or volumes unless clearly intended.
8. Do not install system packages automatically unless I explicitly ask.
9. Only create files and folders inside the current working directory.
10. Before making changes, inspect the current directory.
11. After creating files, show me the final tree and explain how to run it.

README.md should explain clearly:

- What this project is
- That this is a VPS-style server running on my Arch laptop
- That it is not a cloud VPS
- How to install Docker on Arch
- How to create the Docker network
- How to start the Caddy proxy
- How to stop the Caddy proxy
- How to restart the Caddy proxy
- How to check running containers
- How to add a backend project
- How to add a frontend project
- How to use local domains with /etc/hosts
- How to configure real public domains with DNS A records
- How to configure Cloudflare DNS
- How to check my public IP
- How to compare public IP with router WAN IP
- How to detect possible CGNAT
- How to configure router port forwarding
- How to use Cloudflare Tunnel as fallback
- How to avoid exposing database ports publicly
- How to move this setup to a real VPS later

The README should include this public DNS A record architecture:

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

The README should include this Cloudflare Tunnel architecture:

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

The README should explain that public DNS A record mode only works if:

- My laptop is ON
- My laptop does not sleep
- Docker is running
- Caddy is running
- Backend containers are running
- My laptop has a static LAN IP
- My router forwards ports 80 and 443 to my laptop
- My Cloudflare DNS A record points to my home public IPv4
- My ISP does not block ports 80 and 443
- My ISP is not using CGNAT

Create this Caddy Docker Compose file:

proxy/docker-compose.yml

Use:
- image: caddy:2-alpine
- container_name: arch-vps-caddy
- restart: unless-stopped
- ports 80:80 and 443:443
- mount ./Caddyfile to /etc/caddy/Caddyfile:ro
- use persistent named volumes caddy_data and caddy_config
- connect to external network arch-vps-net

Create initial proxy/Caddyfile:

:80 {
    respond "Arch VPS Server Lab is running"
}

Create scripts:

1. scripts/create-network.sh
   - Create Docker network arch-vps-net if it does not already exist.
   - Do not fail if it already exists.

2. scripts/start-proxy.sh
   - Go to proxy directory.
   - Start Caddy with docker compose up -d.

3. scripts/stop-proxy.sh
   - Go to proxy directory.
   - Stop Caddy with docker compose down.
   - Do not remove volumes.

4. scripts/restart-proxy.sh
   - Go to proxy directory.
   - Restart Caddy safely.

5. scripts/list-containers.sh
   - Show running Docker containers.
   - Show all containers.

6. scripts/check-public-ip.sh
   - Show public IP using curl ifconfig.me if curl exists.
   - Show local LAN IPs using ip addr or hostname -I.
   - Print instructions telling me to compare public IP with router WAN IP.
   - Explain: if public IP and router WAN IP are different, ISP may be using CGNAT.

7. scripts/add-local-domain-example.sh
   - Do not edit /etc/hosts automatically.
   - Print example lines to add manually:
     127.0.0.1 archvps.local
     127.0.0.1 api.localhost.test
     127.0.0.1 app.localhost.test
     127.0.0.1 admin.localhost.test

8. scripts/update-project.sh
   - Accept a project folder path as argument.
   - Run git pull.
   - Run docker compose up -d --build.
   - Include error handling if path is missing or does not contain docker-compose.yml / compose.yml.

Add an example backend docker-compose.yml snippet in README.md:

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

Add an example frontend docker-compose.yml snippet in README.md.

Add example Caddy configs in README.md:

Local domain example:

api.localhost.test {
    reverse_proxy my-backend:3000
}

Real public domain example:

api.yourdomain.com {
    reverse_proxy my-backend:3000
}

app.yourdomain.com {
    reverse_proxy my-frontend:8080
}

Root domain example:

yourdomain.com {
    reverse_proxy my-website:8080
}

Add Cloudflare DNS A record examples in README.md:

Type: A
Name: api
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only

Type: A
Name: app
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only

Type: A
Name: @
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only

Add router port forwarding examples in README.md:

Public port 80  -> LAPTOP_STATIC_LAN_IP:80
Public port 443 -> LAPTOP_STATIC_LAN_IP:443

Add security notes in README.md:

- Do not expose PostgreSQL, MySQL, MongoDB, Redis, or other database ports publicly.
- Prefer expose instead of ports for internal backend/database services.
- Use .env files for secrets.
- Do not commit .env files.
- Keep Docker images updated.
- Keep Arch updated.
- Use strong passwords.
- Back up databases.
- Be careful because this exposes my laptop to the internet.

Add .gitignore with:

.env
*.env
*.log
logs/
backups/
data/
node_modules/
dist/
build/
.cache/
.DS_Store

Make all scripts executable.

Final output:
- Show created files.
- Show final folder tree.
- Show commands to run:

./scripts/create-network.sh
./scripts/start-proxy.sh
curl http://localhost

Expected curl output:

Arch VPS Server Lab is running

Also show next steps for real domain:

1. Run ./scripts/check-public-ip.sh
2. Set laptop static LAN IP in router
3. Forward router ports 80 and 443 to laptop
4. Add Cloudflare DNS A records pointing to home public IP
5. Add real domain blocks in proxy/Caddyfile
6. Restart proxy with ./scripts/restart-proxy.sh