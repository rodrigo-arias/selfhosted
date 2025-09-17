# Plexarr

Self-hosted **Plex + Servarr stack** with Docker:

- **Plex** (media server)  
- **qBittorrent** (download client)  
- **Prowlarr** (indexer manager)  
- **Sonarr** (TV shows)  
- **Radarr** (movies)  
- **Bazarr** (subtitles)  
- **Nginx Proxy Manager** (clean LAN URLs)  
- **Portainer** (container management)  
- **Tautulli** (Plex activity & alerts)  

> LAN-only design: no exposed ports. For remote access, use VPN or selectively publish with NPM + SSL.

---

## 📁 Folder structure

```
plexarr/
├─ docker-compose.yml
├─ npm/               # Nginx Proxy Manager data
├─ portainer/         # Portainer data
├─ plex/              # Plex config
├─ qbittorrent/       # qBittorrent config
├─ prowlarr/          # Prowlarr config
├─ sonarr/            # Sonarr config
├─ radarr/            # Radarr config
├─ bazarr/            # Bazarr config
├─ tautulli/          # Tautulli config
└─ data/
   ├─ movies/         # movies library
   ├─ tv/             # TV shows library
   ├─ downloads/      # downloads (qBittorrent)
   └─ transcode/      # transcode temp folder (SSD recommended)
```

> Keep everything under `/data` → allows **hardlinks** (instant moves without duplication).

---

## 🚀 Quick start

1. Clone/create project folder:  
```bash
git clone <your-repo> plexarr
cd plexarr
```

2. Create data folders:  
```bash
mkdir -p data/movies data/tv data/downloads data/transcode
```

3. Review volume paths in `docker-compose.yml` if needed.  

4. Launch stack:  
```bash
docker compose up -d
```

5. Access services (default ports):  
- NPM: `http://localhost:81`  
- Portainer: `http://localhost:9000`  
- Plex: `http://localhost:32400/web`  
- qBittorrent: `http://localhost:8080`  
- Prowlarr: `http://localhost:9696`  
- Sonarr: `http://localhost:8989`  
- Radarr: `http://localhost:7878`  
- Bazarr: `http://localhost:6767`  
- Tautulli: `http://localhost:8181`  

> When using `.lan` domains, configure them via NPM and `/etc/hosts` or a local DNS server.

---

## 🔑 Nginx Proxy Manager (NPM)

Access the NPM admin panel at:

👉 `http://localhost:81`

Default credentials:

- **Email**: `admin@example.com`  
- **Password**: `changeme`

On first login, NPM will force you to:  
1. Set a real email address.  
2. Change the admin password.  

### ⚙️ Adding Proxy Hosts

- Domain: e.g. `plex.lan`  
- Scheme: `http`  
- Forward Hostname/IP: container name (e.g. `plex`, `sonarr`, `radarr`)  
- Forward Port: internal service port (`32400` for Plex, `8989` Sonarr, etc.)  
- Options:  
  - **Websockets**: enable for Plex.  
  - **Block Common Exploits**: disable for Plex, enable for others.  

---

## 🌐 LAN domains with NPM

Examples:

- `plex.lan` → `plex:32400`  
- `sonarr.lan` → `sonarr:8989`  
- `radarr.lan` → `radarr:7878`  
- `prowlarr.lan` → `prowlarr:9696`  
- `torrent.lan` → `qbittorrent:8080`  
- `bazarr.lan` → `bazarr:6767`  
- `tautulli.lan` → `tautulli:8181`  
- `portainer.lan` → `portainer:9000`  

Update `/etc/hosts` to resolve domains:

```
192.168.x.x plex.lan sonarr.lan radarr.lan prowlarr.lan torrent.lan bazarr.lan tautulli.lan portainer.lan npm.lan
```

---

## 🔄 Workflow

- **Prowlarr** → configure indexers (public/private), sync to Sonarr/Radarr.  
- **Sonarr/Radarr**  
  - Root Folders: `/tv`, `/movies`  
  - Downloads: `/downloads`  
  - Quality Profiles: define min/preferred/max (e.g. 1080p WEB-DL → 4K Remux).  
  - Enable **Hardlinks / Atomic Moves**.  
- **qBittorrent**  
  - Scheduler: downloads only from 00:00–07:00 (optional).  
  - Adjust connections and disk cache (256–512 MB).  
  - Default username: `admin`.  
  - First run: a **temporary password** is shown in logs. Retrieve it with:  
    ```bash
    docker logs qbittorrent
    ```  
  - After logging in, change password in: *Tools → Options → Web UI → Authentication*.  
- **Bazarr**  
  - Languages: `eng`, `spa`.  
  - Integrate with Radarr (7878) and Sonarr (8989) using their API Keys.  
  - Enable providers (opensubtitles, subscene, etc.).  
  - Place subtitles next to video files.  
- **Plex**  
  - Libraries: `/movies`, `/tv`.  
  - Scheduled library scans at night.  
  - Transcode directory: `/transcode` (SSD).  
  - Prefer **Direct Play** (Infuse on Apple TV recommended).  
- **Tautulli**  
  - Connect to Plex and set alerts (Telegram, Discord, email).  

---

## 📡 Recommended indexers

With only RARBG (closed) and YTS you won’t see many results. Add more in Prowlarr:

### Public
- **1337x**  
- **EZTV** (TV)  
- **LimeTorrents**  
- **Nyaa** (anime)  

### Private (if invited)
- **IPTorrents**  
- **HDBits**  
- **PassThePopcorn**  
- **BroadcasTheNet**  

> In Prowlarr: *Indexers → Add Indexer* → search and add. Then sync to Radarr/Sonarr.

---

## 🔒 Security

- Designed **LAN-only**. Do **not** expose ports directly.  
- For remote access → VPN (WireGuard) recommended.  
- If publishing Plex/NPM externally → use HTTPS + Access Lists.

---

## 🛠️ Useful tasks

- **Update containers**:  
```bash
docker compose pull
docker compose up -d
```

- **Logs**:  
```bash
docker compose logs -f <service>
```

- **Backup configs**: copy `./plex/config`, `./sonarr/config`, `./npm/data`, etc.

---

## ⚠️ Common issues

- **Slow transcode** → ensure `/transcode` is on SSD.  
- **Slow moves** → check all apps see the same `/data` root (hardlinks).  
- **4K buffering** → server should be wired (Ethernet preferred).  
- **Startup order** → `depends_on` included for sensible order (not health checks).  
