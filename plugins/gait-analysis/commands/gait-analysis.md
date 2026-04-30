---
description: Install (if needed) and run the `gait-analysis` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /gait-analysis — Gait Analysis

Cognitum cog: **Gait Analysis**

Detects walking problems and scores fall risk

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `gait-analysis` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"gait-analysis"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/gait-analysis/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/gait-analysis/logs?lines=5`) and report.

## Usage

```
/gait-analysis
/gait-analysis --once          # one-shot via /console with --once
/gait-analysis --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/gait-analysis --stop           # stop the cog on the seed
/gait-analysis --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"gait-analysis"}`
- `POST /api/v1/apps/gait-analysis/start`
- `POST /api/v1/apps/gait-analysis/stop`
- `POST /api/v1/apps/gait-analysis/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/gait-analysis/logs?lines=N`
- `GET  /api/v1/apps/gait-analysis/manifest`
- `GET  /api/v1/apps/gait-analysis/config`
- `PUT  /api/v1/apps/gait-analysis/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/gait-analysis/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/gait-analysis/`
