# Nextcloud with Traefik Setup

This repository contains the Docker Compose configuration for running Nextcloud behind a Traefik reverse proxy.

## Setup Instructions

1. Clone this repository:
   ```bash
   # Using HTTPS
   git clone https://github.com/mikkelkrogsholm/nextcloud-traefik.git
   # Or using SSH
   git clone git@github.com:mikkelkrogsholm/nextcloud-traefik.git
   ```

2. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```
3. Edit the `.env` file with your specific values:
   - `ACME_EMAIL`: Your email for Let's Encrypt certificates
   - `MYSQL_ROOT_PASSWORD`: A secure password for MySQL root user
   - `MYSQL_PASSWORD`: A secure password for the Nextcloud database user
   - `NEXTCLOUD_ADMIN_USER`: Your desired Nextcloud admin username
   - `NEXTCLOUD_ADMIN_PASSWORD`: A secure password for the Nextcloud admin
   - `NEXTCLOUD_DOMAIN`: Your domain name (e.g., nextcloud.example.com)
   - `NEXTCLOUD_PORT`: The port Nextcloud runs on (usually 80)

4. Start the services:
   ```bash
   docker compose up -d
   ```

## Important Configuration Steps

### Fixing Login Hang Issue

If you experience a hang during login (where the page hangs and requires a refresh to complete login), you need to configure Nextcloud to properly handle the reverse proxy. Run these commands:

```bash
# Add Traefik as a trusted proxy
docker exec -u 33 nextcloud php occ config:system:set trusted_proxies 0 --value="traefik"

# Set the protocol to HTTPS
docker exec -u 33 nextcloud php occ config:system:set overwriteprotocol --value="https"

# Set the correct domain for CLI operations
docker exec -u 33 nextcloud php occ config:system:set overwrite.cli.url --value="https://your-domain.com"

# Set the host that should be used for redirects
docker exec -u 33 nextcloud php occ config:system:set overwritehost --value="your-domain.com"

# Restart the Nextcloud container
docker compose restart nextcloud
```

Replace `your-domain.com` with your actual domain name.

### Why These Settings Are Important

- `trusted_proxies`: Tells Nextcloud to trust headers from Traefik
- `overwriteprotocol`: Ensures Nextcloud knows it's being served over HTTPS
- `overwrite.cli.url`: Sets the base URL for CLI operations and redirects
- `overwritehost`: Ensures correct host headers for redirects

These settings are crucial for proper handling of authentication and redirects when Nextcloud is running behind a reverse proxy. 