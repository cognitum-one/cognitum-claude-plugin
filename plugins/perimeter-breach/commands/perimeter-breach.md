---
description: Install (if needed) and run the `perimeter-breach` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /perimeter-breach — Perimeter Breach

Cognitum cog: **Perimeter Breach**

Guards multiple zones and shows entry direction

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `perimeter-breach` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"perimeter-breach"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/perimeter-breach/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/perimeter-breach/logs?lines=5`) and report.

## Usage

```
/perimeter-breach
/perimeter-breach --once          # one-shot via /console with --once
/perimeter-breach --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/perimeter-breach --stop           # stop the cog on the seed
/perimeter-breach --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"perimeter-breach"}`
- `POST /api/v1/apps/perimeter-breach/start`
- `POST /api/v1/apps/perimeter-breach/stop`
- `POST /api/v1/apps/perimeter-breach/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/perimeter-breach/logs?lines=N`
- `GET  /api/v1/apps/perimeter-breach/manifest`
- `GET  /api/v1/apps/perimeter-breach/config`
- `PUT  /api/v1/apps/perimeter-breach/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `2` | Seconds between perimeter scans |
| `threshold` | float | `3.0` | Z-score threshold to trigger zone breach detection |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/perimeter-breach/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/perimeter-breach/`
