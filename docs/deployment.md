# Deployment

> Prerequisite: complete [setup](setup.md) first — Docker installed, `.env` configured, and the `proxy` Docker network created.

The repo ships with `gaia`, an interactive CLI that wraps Docker Compose across the stacks. Manual deployment with plain `docker compose` is supported as a fallback.

## Using gaia (recommended)

`gaia` is a bash script at the repo root that opens an interactive menu for `up`, `down`, `restart`, `logs`, and `status` actions on stacks or individual services.

### Install (one-time)

Requires bash 4+. From the repo root on the host:

```bash
chmod +x gaia
./gaia install
source ~/.bashrc
```

This downloads [gum](https://github.com/charmbracelet/gum) into `.tools/`, creates a `.tools/gaia` symlink, and adds `.tools/` to your `PATH`. After that, `gaia` is available from any directory.

### Usage

```bash
gaia            # open the interactive menu
gaia install    # re-run setup (idempotent)
gaia help       # show usage
```

The menu prompts for:

1. **Action** — Up, Down, Restart, Logs, Status.
2. **Target** — all stacks, a single stack, or an individual service. Services are auto-discovered from each stack's `docker-compose.yml`.
3. **Confirmation** for destructive actions (`Down`, `Restart`).

## Manual deploy

If you'd rather not use gaia, deploy each stack directly:

```bash
docker network create proxy   # only on first setup

cd infra && docker compose --env-file ../.env up -d
cd ../downloads && docker compose --env-file ../.env up -d
```
