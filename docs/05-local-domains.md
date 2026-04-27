# Local Domains

Use local domains when testing apps on your own laptop.

The helper script prints example `/etc/hosts` entries. It does not edit your system files automatically.

```bash
./scripts/add-local-domain-example.sh
```

Example `/etc/hosts` lines:

```text
127.0.0.1 archvps.local
127.0.0.1 api.localhost.test
127.0.0.1 app.localhost.test
127.0.0.1 admin.localhost.test
```

Example Caddy config:

```caddyfile
api.localhost.test {
    reverse_proxy my-backend:3000
}
```

Restart Caddy after editing:

```bash
./scripts/restart-proxy.sh
```
