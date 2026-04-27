# Cloudflare Tunnel Mode

Cloudflare Tunnel is the fallback when normal inbound traffic cannot reach your laptop.

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

Use this mode when:

- Your ISP uses CGNAT
- Your router cannot forward ports
- Your ISP blocks ports 80 or 443
- You do not want to expose your home public IP directly

The reserved folder is:

```text
tunnels/cloudflare/
```

A common route is:

```text
https://api.yourdomain.com -> http://localhost:80
```

Caddy can still route by hostname to the correct Docker container.
