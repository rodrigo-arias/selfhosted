# Downloads Stack

This stack handles download automation:

- **qBittorrent** → torrent client  
- **Prowlarr** → indexer manager  
- **Sonarr** → TV shows  
- **Radarr** → movies  
- **Bazarr** → subtitles  

---

## Services
- **qBittorrent** → http://torrent.lan (8080)  
  - Username: `admin`  
  - First run: password shown in logs → `docker logs qbittorrent`  
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
1. **Prowlarr** → Add indexers (public/private).  
2. Sync to Sonarr/Radarr.  
3. Sonarr/Radarr send torrents to qBittorrent.  
4. Bazarr downloads subtitles for managed media.  
