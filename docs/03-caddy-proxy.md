# Caddy Proxy

Caddy is the reverse proxy for this setup.

The Compose file lives here:

```text
proxy/docker-compose.yml
```

The Caddy config lives here:

```text
proxy/Caddyfile
```

## Create Network

```bash
./scripts/create-network.sh
```

This creates `arch-vps-net` only if it does not already exist.

## Start Proxy

```bash
./scripts/start-proxy.sh
```

Test:

```bash
curl http://localhost
```

Expected:

```text
Arch VPS Server Lab is running
```

## Stop Proxy

```bash
./scripts/stop-proxy.sh
```

This stops the proxy stack without removing named volumes.

## Restart Proxy

```bash
./scripts/restart-proxy.sh
```

Use this after editing `proxy/Caddyfile`.

## Check Containers

```bash
./scripts/list-containers.sh
```
