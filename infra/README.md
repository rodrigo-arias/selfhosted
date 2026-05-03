# Infra Stack

Reverse proxy, DNS, and uptime monitoring. See the main [README](../README.md) for the service overview.

## First-run configuration

### Nginx Proxy Manager

NPM's admin credentials, proxy hosts, and SSL certificates live in its internal database under `${APPDATA_ROOT}/infra/npm/data/`. They must be configured manually after first start.

1. Access the UI on port `81` (e.g. `http://<NAS_IP>:81`).
2. Default login: `admin@example.com` / `changeme`. Change credentials on first login.
3. Issue a wildcard SSL cert under SSL → Certificates: request a Let's Encrypt certificate for `<your-domain>` and `*.<your-domain>` via DNS challenge with Cloudflare. Requires a Cloudflare API token scoped to "Edit zone DNS" for the domain.

   This assumes the domain is managed by Cloudflare. For other DNS providers (Route 53, DigitalOcean, etc.), NPM ships challenge plugins for them — pick the matching one in the certificate flow and provide the corresponding API credentials.
4. Add proxy hosts for the services you want exposed. Typical settings:

   | Field                  | Value                                                  |
   |------------------------|--------------------------------------------------------|
   | Domain Names           | `<service>.<your-domain>`                              |
   | Scheme                 | `http`                                                 |
   | Forward Hostname       | container name (e.g. `pihole`, `radarr`, `gluetun`)    |
   | Forward Port           | container's internal port                              |
   | Cache Assets           | Off                                                    |
   | Block Common Exploits  | On                                                     |
   | Websockets Support     | On for services with live UIs (Uptime Kuma, Sonarr/Radarr) |

   For services that share Gluetun's network namespace (qBittorrent, gluetun-port-manager), the Forward Hostname is `gluetun` — those services don't have their own network interface. See the network topology diagram in [downloads/README.md](../downloads/README.md).

### Pi-hole

The admin password lives only in the Pi-hole config volume (`${APPDATA_ROOT}/infra/pihole/etc-pihole/`) — set it after first start.

1. Set the admin password inside the container:

   ```bash
   docker exec -it pihole pihole setpassword
   ```

2. Point your router's or clients' DNS to the NAS IP to enable LAN-wide filtering and resolution of `*.<your-domain>` (configured via the `FTLCONF_misc_dnsmasq_lines` env var in the compose file).
