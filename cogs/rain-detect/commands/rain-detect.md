---
description: Install (if needed) and run the `rain-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /rain-detect — Rain Detection

Cognitum cog: **Rain Detection**

Detects when rain starts, stops, and how heavy it is

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `rain-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"rain-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/rain-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/rain-detect/logs?lines=5`) and report.

## Usage

```
/rain-detect
/rain-detect --once          # one-shot via /console with --once
/rain-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/rain-detect --stop           # stop the cog on the seed
/rain-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"rain-detect"}`
- `POST /api/v1/apps/rain-detect/start`
- `POST /api/v1/apps/rain-detect/stop`
- `POST /api/v1/apps/rain-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/rain-detect/logs?lines=N`
- `GET  /api/v1/apps/rain-detect/manifest`
- `GET  /api/v1/apps/rain-detect/config`
- `PUT  /api/v1/apps/rain-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `30` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/rain-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/rain-detect/`
