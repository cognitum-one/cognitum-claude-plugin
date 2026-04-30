---
description: Install (if needed) and run the `shelf-engagement` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /shelf-engagement — Shelf Engagement

Cognitum cog: **Shelf Engagement**

Detects when customers interact with products

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `shelf-engagement` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"shelf-engagement"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/shelf-engagement/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/shelf-engagement/logs?lines=5`) and report.

## Usage

```
/shelf-engagement
/shelf-engagement --once          # one-shot via /console with --once
/shelf-engagement --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/shelf-engagement --stop           # stop the cog on the seed
/shelf-engagement --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"shelf-engagement"}`
- `POST /api/v1/apps/shelf-engagement/start`
- `POST /api/v1/apps/shelf-engagement/stop`
- `POST /api/v1/apps/shelf-engagement/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/shelf-engagement/logs?lines=N`
- `GET  /api/v1/apps/shelf-engagement/manifest`
- `GET  /api/v1/apps/shelf-engagement/config`
- `PUT  /api/v1/apps/shelf-engagement/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/shelf-engagement/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/shelf-engagement/`
