# Infra Stack

Reverse proxy, DNS, and uptime monitoring. See the main [README](../README.md) for the service overview.

## First-run configuration

### Nginx Proxy Manager

NPM's admin credentials, proxy hosts, and SSL certificates live in its internal database under `${APPDATA_ROOT}/infra/npm/data/`. They must be configured manually after first start.

1. Access the UI on port `81` (e.g. `http://<LAN_IP>:81`).
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

### Homepage

YAML files in `${APPDATA_ROOT}/infra/homepage/config/`, hot-reloaded on save. Create what you need; missing files fall back to defaults:

- `settings.yaml` — title, theme, section layout.
- `services.yaml` — service link cards grouped by section (name, icon, `href`).
- `widgets.yaml` — top-of-page widgets. Useful: `resources` (host CPU/RAM/disk/temp), `search`, and `uptimekuma` pointing at `http://uptime-kuma:3001`.
- `bookmarks.yaml` — optional.

In NPM, add a proxy host for the apex domain (`<your-domain>`) → `homepage:3000`. Homepage rejects requests whose `Host` isn't in `HOMEPAGE_ALLOWED_HOSTS` — the compose file passes `${DOMAIN}`; extend it (comma-separated) if you serve it elsewhere too.

The Docker socket is intentionally **not** mounted: Uptime Kuma already covers live status, and skipping it avoids exposing container env vars to the dashboard.

### Ntfy

Drop `server.yml` at `${APPDATA_ROOT}/infra/ntfy/etc/server.yml`. Minimum config, with ntfy.sh as the iOS push relay:

```yaml
base-url: "https://alerts.<your-domain>"
upstream-base-url: "https://ntfy.sh"
cache-file: "/var/cache/ntfy/cache.db"
cache-duration: "12h"
attachment-cache-dir: "/var/cache/ntfy/attachments"
behind-proxy: true
```

`upstream-base-url` is what enables iOS notifications without your own APNs integration — ntfy forwards a poll to ntfy.sh, which pings the iPhone via Apple Push. `behind-proxy` makes ntfy trust `X-Forwarded-*` from NPM.

In NPM, add a proxy host for `alerts.<your-domain>` → `ntfy:80` with **Websockets Support: On** (web UI and `--subscribe` need it). Don't set `proxy_buffering off` in the proxy host's Advanced config — it breaks SSL termination, and ntfy's SSE works fine through NPM's default buffering.

The container starts fine without `server.yml` (defaults, port 80), so the proxy host works immediately — adding the file just switches on the relay.

For receiving notifications as native banners on macOS via the ntfy CLI + a LaunchAgent, see [docs/mac-setup.md](../docs/mac-setup.md).

### Cloudflared

Tunnel-as-a-service: all routing lives in the Cloudflare Zero Trust dashboard.

1. **Networks → Tunnels → Create a tunnel**, connector "Cloudflared", name it (e.g. `homelab`), copy the token from the install command.
2. Set `CLOUDFLARE_TUNNEL_TOKEN=...` in `.env` and start the container.
3. Under the tunnel's **Public Hostname** tab, add each subdomain you want exposed. Point the service at `https://npm:443` with **No TLS Verify** so NPM still terminates SSL and applies its proxy host rules.
4. To gate a public route, create an **Access** application for the hostname (one-time PIN by email is free up to 50 users).
5. For programmatic clients (mobile apps, scripts) that can't do the interactive PIN flow, create a **Service Token** under Access → Service Auth. Add a policy to the Access app that includes the Service Token, and place it **above** the email-PIN policy — Access matches top-down. The client then sends `CF-Access-Client-Id` and `CF-Access-Client-Secret` headers on each request to bypass the PIN.
