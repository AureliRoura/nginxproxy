# nginxproxy

A self-hosted home server dashboard running on a local network. Nginx acts as a reverse proxy and serves a visual dashboard to access all services from a single entry point on port 80.

## Structure

```
docker-compose.yaml
nginx/
    nginx.conf       # Nginx reverse proxy configuration
    index.html       # Dashboard UI
```

## Services

| Path | Service | Port | Method |
|---|---|---|---|
| `/portainer/` | Portainer (Docker UI) | 9000 | redirect |
| `/openhab/` | OpenHAB (home automation) | 8080 | redirect |
| `/nextcloud/` | Nextcloud (file storage) | 8090 | redirect |
| `/torrent/` | Torrent client | 8091 | redirect |
| `/duplicati/` | Duplicati (backups) | 8300 | redirect |
| `/menus/` | Custom menus app | 9092 | redirect |
| `/frontail/` | Frontail (log viewer) | 9001 | proxy rewrite |
| `/zigbee2mqtt/` | Zigbee2MQTT (IoT gateway) | 8081 | proxy rewrite |

## Routing strategies

- **`return 301` redirect** — browser is redirected directly to the service's port. Simple but changes the URL in the browser.
- **`rewrite` + `proxy_pass`** — request is proxied server-side with the sub-path stripped. The user stays on port 80 and the original URL is preserved.

## Setup

### Prerequisites

- Docker and Docker Compose installed on the host
- Services running on `192.168.2.9` at their respective ports

### Deploy

Copy the config files to the host:

```bash
cp nginx/nginx.conf /opt/nginx/nginx.conf
cp nginx/index.html /opt/nginx/index.html
```

Start the container:

```bash
docker compose up -d
```

The dashboard will be available at `http://192.168.2.9`.

### Update configuration

The container mounts files directly from `/opt/nginx/` on the host. After editing `nginx.conf` or `index.html` in this repo, copy the updated files to the host mount path and reload nginx — no container restart needed:

```bash
cp nginx/nginx.conf /opt/nginx/nginx.conf
cp nginx/index.html /opt/nginx/index.html
docker exec nginx-proxy nginx -s reload
```

If you change `docker-compose.yaml` itself (e.g. ports, volumes, image), recreate the container:

```bash
docker compose up -d --force-recreate
```
