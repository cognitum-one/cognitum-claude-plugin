---
description: Install (if needed) and run the `dwell-heatmap` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /dwell-heatmap — Dwell Heatmap

Cognitum cog: **Dwell Heatmap**

Shows where customers spend the most time

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `dwell-heatmap` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"dwell-heatmap"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/dwell-heatmap/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/dwell-heatmap/logs?lines=5`) and report.

## Usage

```
/dwell-heatmap
/dwell-heatmap --once          # one-shot via /console with --once
/dwell-heatmap --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/dwell-heatmap --stop           # stop the cog on the seed
/dwell-heatmap --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"dwell-heatmap"}`
- `POST /api/v1/apps/dwell-heatmap/start`
- `POST /api/v1/apps/dwell-heatmap/stop`
- `POST /api/v1/apps/dwell-heatmap/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/dwell-heatmap/logs?lines=N`
- `GET  /api/v1/apps/dwell-heatmap/manifest`
- `GET  /api/v1/apps/dwell-heatmap/config`
- `PUT  /api/v1/apps/dwell-heatmap/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/dwell-heatmap/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/dwell-heatmap/`
