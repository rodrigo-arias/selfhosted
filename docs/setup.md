# Setup

## Prerequisites

- Docker Engine and Docker Compose v2.
- A reachable domain (configured in public DNS or resolved locally via Pi-hole) pointing to the host running this stack.
- The `proxy` Docker network, created once before the first deploy:

```bash
docker network create proxy
```

## Environment variables

Copy `.env.example` to `.env` and fill in:

| Variable | Description |
|---|---|
| `PUID` / `PGID` | User/group IDs for container processes (typically `1000`/`1000`) |
| `TZ` | Timezone (e.g. `America/Argentina/Buenos_Aires`) |
| `APPDATA_ROOT` | Path for container application data |
| `MEDIA_ROOT` | Path for media files (movies, TV, downloads) |
| `LAN_IP` | Host LAN IP — used for direct port bindings and local DNS |
| `DOMAIN` | Domain used for reverse-proxied services (`*.<DOMAIN>`) |
| `VPN_SERVICE_PROVIDER`, `VPN_WIREGUARD_PRIVATE_KEY`, `VPN_WIREGUARD_ADDRESS`, `VPN_COUNTRY` | Gluetun VPN config for the downloads stack |
| `QBITTORRENT_USER` / `QBITTORRENT_PASS` | qBittorrent credentials |

## Required folders

Create the media subfolders under `MEDIA_ROOT` before the first deploy:

```bash
mkdir -p "${MEDIA_ROOT}/movies"
mkdir -p "${MEDIA_ROOT}/tv"
mkdir -p "${MEDIA_ROOT}/downloads"
```

`APPDATA_ROOT` subfolders are created automatically by containers on first run.
