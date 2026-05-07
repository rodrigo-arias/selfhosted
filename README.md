# Selfhosted

A self-hosted homelab stack on Docker Compose, split into **infra** and **media**. Services share an external `proxy` network and are reverse-proxied through **Nginx Proxy Manager** at `*.<your-domain>` on the LAN.

## Quickstart

1. Install prerequisites and configure `.env` — see [docs/setup.md](docs/setup.md).
2. Deploy with the bundled `gaia` CLI or manually — see [docs/deployment.md](docs/deployment.md).

## Structure

```
selfhosted/
│
├── infra/        # Reverse proxy, DNS, monitoring
├── media/        # VPN-gated torrent client + *arr stack
├── data/         # Shared media (movies, tv, downloads)
├── docs/         # Setup and deployment guides
├── gaia          # Interactive deploy CLI
└── .env          # Environment variables (gitignored; see .env.example)
```

Each stack folder has its own `docker-compose.yml` and a stack-specific README.

## Services

Subdomains map to services via proxy hosts configured in Nginx Proxy Manager. The names below are the ones used in this deploy — pick your own when setting up NPM.

| Subdomain                  | Service              | Stack      |
|----------------------------|----------------------|------------|
| `<your-domain>`            | Homepage (dashboard) | infra      |
| `proxy.<your-domain>`      | Nginx Proxy Manager  | infra      |
| `dns.<your-domain>`        | Pi-hole              | infra      |
| `status.<your-domain>`     | Uptime Kuma          | infra      |
| `alerts.<your-domain>`     | Ntfy                 | infra      |
| `downloads.<your-domain>`  | qBittorrent (via Gluetun VPN) | media |
| `indexer.<your-domain>`    | Prowlarr             | media      |
| `tv.<your-domain>`         | Sonarr               | media      |
| `movies.<your-domain>`     | Radarr               | media      |
| `subs.<your-domain>`       | Bazarr               | media      |

Cloudflared runs in the infra stack as well but has no UI — it opens an outbound tunnel to Cloudflare so selected subdomains can be reached from the public internet behind Cloudflare Access.
