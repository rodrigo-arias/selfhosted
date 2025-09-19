# Infra Stack

This stack includes infrastructure and utility services:

- **Nginx Proxy Manager (NPM)** → reverse proxy, LAN `.lan` domains  
- **Pi-hole** → DNS + ad/tracker blocking  
- **Uptime Kuma** → uptime and Internet monitoring  

---

## Configuration

### Nginx Proxy Manager (NPM)

1. Access the NPM UI → http://localhost:81/login  
   - Default login: `admin@example.com / changeme`  
   - Change credentials on first login.

2. Add proxy hosts for infra services:

| Domain       | Scheme | Forward Hostname | Forward Port | Cache Assets | Websockets | Block Common Exploits |
|--------------|--------|------------------|--------------|--------------|------------|------------------------|
| `pihole.lan` | http   | pihole           | 80           | Off          | Off        | On                     |
| `kuma.lan`   | http   | uptime-kuma      | 3001         | Off          | On         | On                     |
| `npm.lan`    | http   | npm              | 81           | Off          | Off        | On                     |

*(add more services from other stacks as needed)*

---

### Pi-hole

1. Set a password manually inside the container:

```bash
docker exec -it pihole pihole setpassword
```
- Access Pi-hole at http://pihole.lan (via NPM).
- Point your router’s DNS server to the NAS IP to enable filtering for your LAN.

---