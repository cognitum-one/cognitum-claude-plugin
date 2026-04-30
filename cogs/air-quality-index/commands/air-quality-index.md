---
description: Install (if needed) and run the `air-quality-index` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /air-quality-index — Air Quality Monitor

Cognitum cog: **Air Quality Monitor**

Track indoor air quality with CO2 and particle sensors

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `air-quality-index` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"air-quality-index"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/air-quality-index/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/air-quality-index/logs?lines=5`) and report.

## Usage

```
/air-quality-index
/air-quality-index --once          # one-shot via /console with --once
/air-quality-index --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/air-quality-index --stop           # stop the cog on the seed
/air-quality-index --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"air-quality-index"}`
- `POST /api/v1/apps/air-quality-index/start`
- `POST /api/v1/apps/air-quality-index/stop`
- `POST /api/v1/apps/air-quality-index/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/air-quality-index/logs?lines=N`
- `GET  /api/v1/apps/air-quality-index/manifest`
- `GET  /api/v1/apps/air-quality-index/config`
- `PUT  /api/v1/apps/air-quality-index/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `30` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/air-quality-index/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/air-quality-index/`
