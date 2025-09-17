# Infra Stack

This stack includes infrastructure and utility services:

- **Nginx Proxy Manager (NPM)** → reverse proxy, LAN `.lan` domains  
- **Pi-hole** → DNS + ad/tracker blocking  
- **Uptime Kuma** → uptime and Internet monitoring  

---

## Services
- **NPM UI** → http://npm.lan (port 81 internally)
  - Admin panel: http://localhost:81/login
  - Default login: `admin@example.com / changeme`  
  - Change email & password on first login  
- **Pi-hole** → http://pihole.lan (port 80 internal, mapped to host 8081)  
  - Default password: set in `.env` (`PIHOLE_WEBPASSWORD`)  
  - DNS server for LAN → point router DHCP DNS to the NAS  
- **Uptime Kuma** → http://kuma.lan (port 3001)  
  - Add monitors (DNS, ping, HTTP) for LAN and WAN services
