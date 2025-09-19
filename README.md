# Selfhosted

A self-hosted stack of Docker services for home use, split into **infra**, **downloads**, and **streaming**.  
All services are connected to a shared external Docker network called `proxy`, so they can be routed through **Nginx Proxy Manager (NPM)** with friendly `.lan` domains.

---

## Structure

```
selfhosted/
│
├── infra/        # Infrastructure & utilities (NPM, Pi-hole, Uptime Kuma)
├── downloads/    # Media automation & downloads (qBit, *arrs, Bazarr)
├── streaming/    # Plex server & monitoring (Plex, Tautulli)
├── data/         # Shared storage (movies, tv, downloads, transcode)
└── .env          # Environment variables
```

Each subfolder contains its own `docker-compose.yml` and `README.md` with setup instructions.

---

## Services

| Domain         | Service               | Stack      | Port  | Notes                                    |
|----------------|-----------------------|------------|-------|------------------------------------------|
| `npm.lan`      | Nginx Proxy Manager   | infra      | 81    | Reverse proxy & UI                       |
| `pihole.lan`   | Pi-hole               | infra      | 8081  | DNS + ad/tracker blocking                |
| `kuma.lan`     | Uptime Kuma           | infra      | 3001  | Service & Internet monitoring            |
| `torrent.lan`  | qBittorrent           | downloads  | 8080  | Torrent client                           |
| `prowlarr.lan` | Prowlarr              | downloads  | 9696  | Indexer manager                          |
| `sonarr.lan`   | Sonarr                | downloads  | 8989  | TV shows manager                         |
| `radarr.lan`   | Radarr                | downloads  | 7878  | Movies manager                           |
| `bazarr.lan`   | Bazarr                | downloads  | 6767  | Subtitles manager                        |
| `plex.lan`     | Plex Media Server     | streaming  | 32400 | Media server                             |
| `tautulli.lan` | Tautulli              | streaming  | 8181  | Plex monitoring & stats                  |

## Setup

### Environment Variables

Configure the following variables in `.env`:

- `PUID` - User ID for Docker containers (usually 1000)
- `PGID` - Group ID for Docker containers (usually 1000)
- `TZ` - Time zone (e.g., America/Argentina/Buenos_Aires)
- `APPDATA_ROOT` - Path for Docker application data and configurations
- `MEDIA_ROOT` - Path for media files (movies, TV shows, downloads, etc.)
- `NAS_IP` - Fixed IP address of your NAS/server
- `QBIT_WEBUI_USERNAME` - qBittorrent web UI username (default: admin)
- `QBIT_WEBUI_PASSWORD` - qBittorrent web UI password (leave empty for auto-generation)

### Required Folders

Create the following folder structure under `MEDIA_ROOT`:

```bash
# Create media folders
mkdir -p "${MEDIA_ROOT}/movies"     # Movie files
mkdir -p "${MEDIA_ROOT}/tv"         # TV show files  
mkdir -p "${MEDIA_ROOT}/downloads"  # Downloaded content
mkdir -p "${MEDIA_ROOT}/transcode"  # Plex transcoding cache (optional)
```

The `APPDATA_ROOT` folders will be created automatically by Docker containers on first run.

## Deployment

```bash
# Create the shared proxy network (only once)
docker network create proxy || echo "ℹ️ Proxy network already exists"

# Start Infra stack
cd infra
docker compose --env-file ../.env up -d

# Start Downloads stack
cd ../downloads
docker compose --env-file ../.env up -d

# Start Streaming stack
cd ../streaming
docker compose --env-file ../.env up -d

# Return to project root
cd ..
echo "✅ All stacks started"
```