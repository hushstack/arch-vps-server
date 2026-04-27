# Troubleshooting

## `network arch-vps-net declared as external, but could not be found`

Create the shared network:

```bash
./scripts/create-network.sh
```

Then start the proxy again:

```bash
./scripts/start-proxy.sh
```

## Docker Desktop Cannot Mount `/mnt/storage`

If you see an error like:

```text
The path /mnt/storage/... is not shared from the host
```

You are probably using Docker Desktop's `desktop-linux` context.

For this project, use native Linux Docker:

```bash
docker context use default
sudo systemctl enable --now docker
```

Then try again:

```bash
./scripts/create-network.sh
./scripts/start-proxy.sh
```

## Docker Bridge Networking Fails After Arch Update

If Docker cannot create a veth pair or bridge network after a kernel update, reboot:

```bash
sudo reboot
```

After reboot:

```bash
cd /mnt/storage/arch-vps-server
docker context use default
sudo systemctl enable --now docker
./scripts/create-network.sh
./scripts/start-proxy.sh
curl http://localhost
```

## Check Status

```bash
docker context show
docker network ls
docker ps
./scripts/list-containers.sh
```
