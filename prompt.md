I want you to set up this folder as my local Arch VPS-style server lab.

Current working directory:
/mnt/storage/arch-vps-server

Goal:
Create a clean Docker-based local VPS environment where I can clone backend projects, run them with Docker Compose, and route local domains through a Caddy reverse proxy like a real VPS server.

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
│   ├── start-proxy.sh
│   ├── stop-proxy.sh
│   ├── restart-proxy.sh
│   ├── list-containers.sh
│   ├── create-network.sh
│   └── add-local-domain-example.sh
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
   - update /etc/hosts
   - use Cloudflare Tunnel later
9. Add an example backend docker-compose.yml snippet in the README.
10. Add an example Caddy reverse_proxy config in the README.
11. Add a useful .gitignore for env files, logs, backups, and local data.
12. Keep the project clean, simple, and easy to maintain.

Important:
- Do not assume this is a real VPS.
- This is a local VPS-style lab running on my Arch laptop.
- Do not install system packages automatically unless I explicitly ask.
- Only create files and folders inside the current directory.
- Before making changes, inspect the current directory.
- After creating files, show me the final tree and explain how to run it.

Expected commands after setup should be something like:

docker network create arch-vps-net
cd proxy
docker compose up -d

Then I should be able to test:

curl http://localhost

Expected output:

Arch VPS Server Lab is running