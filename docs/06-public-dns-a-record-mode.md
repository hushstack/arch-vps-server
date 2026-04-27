# Public DNS A Record Mode

This mode works like a small home-hosted VPS.

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

## Requirements

Public DNS A record mode only works if:

- Your laptop is ON
- Your laptop does not sleep
- Docker is running
- Caddy is running
- Backend containers are running
- Your laptop has a static LAN IP
- Your router forwards ports 80 and 443 to your laptop
- Your Cloudflare DNS A record points to your home public IPv4
- Your ISP does not block ports 80 and 443
- Your ISP is not using CGNAT

## Check Public IP

```bash
./scripts/check-public-ip.sh
```

Compare:

- Public IP from the script
- Router WAN / Internet IP from your router admin page

If they are different, your ISP may be using CGNAT.

## Router Port Forwarding

```text
Public port 80  -> LAPTOP_STATIC_LAN_IP:80
Public port 443 -> LAPTOP_STATIC_LAN_IP:443
```

## Cloudflare DNS Examples

```text
Type: A
Name: api
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only
```

```text
Type: A
Name: app
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only
```

```text
Type: A
Name: @
Content: YOUR_HOME_PUBLIC_IP
Proxy status: Proxied or DNS only
```

## Caddy Examples

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
