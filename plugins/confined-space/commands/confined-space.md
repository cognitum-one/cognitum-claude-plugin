---
description: Install (if needed) and run the `confined-space` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /confined-space — Confined Space

Cognitum cog: **Confined Space**

Monitors workers in tight spaces for safety

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `confined-space` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"confined-space"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/confined-space/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/confined-space/logs?lines=5`) and report.

## Usage

```
/confined-space
/confined-space --once          # one-shot via /console with --once
/confined-space --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/confined-space --stop           # stop the cog on the seed
/confined-space --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"confined-space"}`
- `POST /api/v1/apps/confined-space/start`
- `POST /api/v1/apps/confined-space/stop`
- `POST /api/v1/apps/confined-space/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/confined-space/logs?lines=N`
- `GET  /api/v1/apps/confined-space/manifest`
- `GET  /api/v1/apps/confined-space/config`
- `PUT  /api/v1/apps/confined-space/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `2` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/confined-space/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/confined-space/`
