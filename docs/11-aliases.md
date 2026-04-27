# Aliases

The aliases file gives you short commands for daily hosting work.

File:

```text
scripts/aliases.sh
```

## Enable For Current Terminal

```bash
source /mnt/storage/arch-vps-server/scripts/aliases.sh
```

Then check available commands:

```bash
avps-help
```

## Enable Automatically

For Bash, add this line to `~/.bashrc`:

```bash
source /mnt/storage/arch-vps-server/scripts/aliases.sh
```

For Zsh, add this line to `~/.zshrc`:

```bash
source /mnt/storage/arch-vps-server/scripts/aliases.sh
```

Restart your terminal after editing the shell config.

## Common Commands

```bash
avps
avps-net
avps-up
avps-down
avps-restart
avps-ps
avps-ip
avps-caddy
avps-logs
```

## Host a Backend Project

Use:

```bash
avps-host-backend DOMAIN CONTAINER_NAME INTERNAL_PORT PROJECT_DIR
```

Example:

```bash
avps-host-backend api.yourdomain.com my-backend 3000 ./projects/my-backend
```

This does not edit files automatically. It prints:

- Backend `docker-compose.yml` snippet
- Caddy reverse proxy block
- Start/rebuild command
- Restart command
- Test command

## Update a Project

```bash
avps-update ./projects/my-backend
```

This runs the existing safe update script:

```bash
git pull
docker compose up -d --build
```
