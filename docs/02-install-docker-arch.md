# Install Docker on Arch

This project does not install packages automatically.

Typical Arch commands:

```bash
sudo pacman -Syu docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
```

After adding your user to the `docker` group, log out and log back in.

Check Docker:

```bash
docker --version
docker compose version
docker context ls
```

If you use Docker Desktop and native Docker together, prefer the native Linux context for this project:

```bash
docker context use default
```

Then start Docker:

```bash
sudo systemctl enable --now docker
```

If Docker bridge networking fails after an Arch update, reboot once so the running kernel matches installed kernel modules:

```bash
sudo reboot
```
