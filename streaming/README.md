# Streaming Stack

This stack provides media streaming and monitoring:

- **Plex** → media server  
- **Tautulli** → Plex activity monitoring & alerts  

---

## Services
- **Plex** → http://plex.lan (32400)  
  - Libraries: `/movies`, `/tv`  
  - Transcode folder: `/transcode` (SSD recommended)  
  - Prefer Direct Play for LAN clients (e.g. Infuse on Apple TV)  
- **Tautulli** → http://tautulli.lan (8181)  
  - Connect to Plex → track activity & send alerts

---

## Notes
- Plex should be **wired (Ethernet)** for 4K Remux playback.  
- Disable “Block Common Exploits” in NPM for Plex (requires WebSockets).  
- Enable it for Tautulli and all other services.  
