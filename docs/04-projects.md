# Projects

Clone backend and frontend apps into `projects/`.

All app containers that Caddy should reach must join the external Docker network:

```text
arch-vps-net
```

## Backend Example

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

Example Caddy route:

```caddyfile
api.localhost.test {
    reverse_proxy my-backend:3000
}
```

## Frontend Example

```yaml
services:
  my-frontend:
    build: .
    container_name: my-frontend
    restart: unless-stopped
    expose:
      - "8080"
    networks:
      - arch-vps-net

networks:
  arch-vps-net:
    external: true
```

Example Caddy route:

```caddyfile
app.localhost.test {
    reverse_proxy my-frontend:8080
}
```

## Update Project

```bash
./scripts/update-project.sh ./projects/my-backend
```

The update script runs:

```bash
git pull
docker compose up -d --build
```

It checks that the path exists, is a Git repo, and contains `docker-compose.yml` or `compose.yml`.
