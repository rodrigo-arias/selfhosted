# Downloads Stack

This stack handles download automation:

- **Gluetun** → VPN client
- **qBittorrent** → torrent client
- **Gluetun Port Manager** → automatic port forwarding
- **Prowlarr** → indexer manager  
- **Sonarr** → TV shows  
- **Radarr** → movies  
- **Bazarr** → subtitles  

---

## Services
- **Gluetun** → VPN tunnel for qBittorrent  
  - Protocol: Wireguard  
  - Port forwarding: Enabled
- **qBittorrent** → http://torrent.lan (8080)  
  - Username: `admin`  
  - First run: password shown in logs → `docker logs qbittorrent`  
  - All torrent traffic routed through VPN
- **Gluetun Port Manager** → Automatic port management
  - Updates qBittorrent with forwarded port from VPN
- **Prowlarr** → http://prowlarr.lan (9696)  
  - Configure indexers and sync with Sonarr/Radarr  
- **Sonarr** → http://sonarr.lan (8989)  
  - Root folder: `/tv`  
  - Downloads: `/downloads`  
- **Radarr** → http://radarr.lan (7878)  
  - Root folder: `/movies`  
  - Downloads: `/downloads`  
- **Bazarr** → http://bazarr.lan (6767)
  - Connect to Sonarr & Radarr using API keys  

---

## Workflow
1. **Gluetun** → Establishes VPN connection with port forwarding
2. **Gluetun Port Manager** → Updates qBittorrent with forwarded port
3. **Prowlarr** → Add indexers (public/private)
4. Sync to **Sonarr/Radarr**
5. **Sonarr/Radarr** send torrents to qBittorrent  
6. **Bazarr** downloads subtitles for managed media  
