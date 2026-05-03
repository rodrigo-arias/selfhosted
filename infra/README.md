# Infra Stack

Reverse proxy, DNS, and uptime monitoring. See the main [README](../README.md) for the service overview.

## First-run configuration

### Nginx Proxy Manager

1. Access the UI on port `81` (e.g. `http://<NAS_IP>:81`).
2. Default login: `admin@example.com` / `changeme`. Change credentials on first login.
3. Add proxy hosts for the services you want exposed. Typical settings:

   | Field                  | Value                                                  |
   |------------------------|--------------------------------------------------------|
   | Domain Names           | `<service>.<your-domain>`                              |
   | Scheme                 | `http`                                                 |
   | Forward Hostname       | container name (e.g. `pihole`, `radarr`, `gluetun`)    |
   | Forward Port           | container's internal port                              |
   | Cache Assets           | Off                                                    |
   | Block Common Exploits  | On                                                     |
   | Websockets Support     | On for services with live UIs (Uptime Kuma, Sonarr/Radarr) |

4. To enable HTTPS with a wildcard cert, configure a DNS challenge for `<your-domain>` under SSL → Certificates.

### Pi-hole

1. Set the admin password inside the container:

   ```bash
   docker exec -it pihole pihole setpassword
   ```

2. Point your router's or clients' DNS to the NAS IP to enable LAN-wide filtering and resolution of `*.<your-domain>` (configured via the `FTLCONF_misc_dnsmasq_lines` env var in the compose file).
