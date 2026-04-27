# Move to a Real VPS

This setup can move to a real VPS later.

Steps:

1. Rent a VPS and install Docker.
2. Copy this folder to the VPS.
3. Create the `arch-vps-net` Docker network.
4. Copy or clone your projects into `projects/`.
5. Copy production `.env` files securely.
6. Start project containers.
7. Start the Caddy proxy.
8. Point Cloudflare DNS records to the VPS public IP instead of your home public IP.
9. Remove home router port forwarding if you no longer need it.

The same Docker Compose and Caddy pattern works on a real VPS with fewer home-network limitations.
