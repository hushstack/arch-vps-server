# Overview

This folder is a Docker-based VPS-style server lab for an Arch laptop.

It lets you clone backend and frontend projects into `projects/`, run them with Docker Compose, connect them to one shared Docker network, and route domains through Caddy.

This is not a cloud VPS. It runs on your laptop, so uptime depends on your laptop, Docker, your home router, your ISP, and your network.

## Main Components

- `proxy/`: Caddy reverse proxy and Compose file.
- `projects/`: cloned backend and frontend apps.
- `databases/`: local database-related folders.
- `tunnels/cloudflare/`: reserved for Cloudflare Tunnel configuration.
- `scripts/`: safe helper scripts.
- `backups/`: database and app backups.
- `logs/`: local logs.

## Shared Network

All public-facing app containers should join this external Docker network:

```text
arch-vps-net
```

Caddy also joins that network, so it can proxy to containers by Docker service or container name.

## Public Access Modes

You can expose real domains in two ways:

- DNS A Record Mode: normal public DNS points to your home public IPv4, then your router forwards ports to the laptop.
- Cloudflare Tunnel Mode: Cloudflare Tunnel connects outbound from your laptop to Cloudflare, useful when direct inbound traffic does not work.
