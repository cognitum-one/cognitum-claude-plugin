---
description: Install (if needed) and run the `anomaly-attractor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /anomaly-attractor — Anomaly Attractor

Cognitum cog: **Anomaly Attractor**

Learns what's normal and catches anything weird

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `anomaly-attractor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"anomaly-attractor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/anomaly-attractor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/anomaly-attractor/logs?lines=5`) and report.

## Usage

```
/anomaly-attractor
/anomaly-attractor --once          # one-shot via /console with --once
/anomaly-attractor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/anomaly-attractor --stop           # stop the cog on the seed
/anomaly-attractor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"anomaly-attractor"}`
- `POST /api/v1/apps/anomaly-attractor/start`
- `POST /api/v1/apps/anomaly-attractor/stop`
- `POST /api/v1/apps/anomaly-attractor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/anomaly-attractor/logs?lines=N`
- `GET  /api/v1/apps/anomaly-attractor/manifest`
- `GET  /api/v1/apps/anomaly-attractor/config`
- `PUT  /api/v1/apps/anomaly-attractor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/anomaly-attractor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/anomaly-attractor/`
