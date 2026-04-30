---
description: Install (if needed) and run the `cough-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /cough-detect — Cough Detection

Cognitum cog: **Cough Detection**

Acoustic transient + spectral cough detector with 30s cluster aggregation. Early-warning signal for respiratory illness

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `cough-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"cough-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/cough-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/cough-detect/logs?lines=5`) and report.

## Usage

```
/cough-detect
/cough-detect --once          # one-shot via /console with --once
/cough-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/cough-detect --stop           # stop the cog on the seed
/cough-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"cough-detect"}`
- `POST /api/v1/apps/cough-detect/start`
- `POST /api/v1/apps/cough-detect/stop`
- `POST /api/v1/apps/cough-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/cough-detect/logs?lines=N`
- `GET  /api/v1/apps/cough-detect/manifest`
- `GET  /api/v1/apps/cough-detect/config`
- `PUT  /api/v1/apps/cough-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `transient_z` | float | `3.0` | Z-score spike threshold to flag a candidate cough |
| `cluster_window` | integer | `30` | Seconds to count clustered events |
| `alert_count` | integer | `3` | Events within window to fire cluster alert |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/cough-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/cough-detect/`
