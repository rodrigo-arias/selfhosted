# Selfhosted

A Docker-based home stack split into **infra** and **downloads**. Services share an external `proxy` Docker network and are reverse-proxied through **Nginx Proxy Manager** at `*.<your-domain>` on the LAN.

## Quickstart

1. Install prerequisites and configure `.env` — see [docs/setup.md](docs/setup.md).
2. Deploy with the bundled `gaia` CLI or manually — see [docs/deployment.md](docs/deployment.md).

## Structure

```
selfhosted/
│
├── infra/        # Reverse proxy, DNS, monitoring
├── downloads/    # VPN-gated torrent client + *arr stack
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
| `proxy.<your-domain>`      | Nginx Proxy Manager  | infra      |
| `dns.<your-domain>`        | Pi-hole              | infra      |
| `status.<your-domain>`     | Uptime Kuma          | infra      |
| `downloads.<your-domain>`  | qBittorrent (via Gluetun VPN) | downloads |
| `indexer.<your-domain>`    | Prowlarr             | downloads  |
| `tv.<your-domain>`         | Sonarr               | downloads  |
| `movies.<your-domain>`     | Radarr               | downloads  |
| `subs.<your-domain>`       | Bazarr               | downloads  |
