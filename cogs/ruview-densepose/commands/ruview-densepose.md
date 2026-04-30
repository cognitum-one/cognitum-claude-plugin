---
description: Install (if needed) and run the `ruview-densepose` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /ruview-densepose — RuView WiFi DensePose

Cognitum cog: **RuView WiFi DensePose**

Full body pose tracking from WiFi — no cameras needed

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `ruview-densepose` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"ruview-densepose"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/ruview-densepose/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/ruview-densepose/logs?lines=5`) and report.

## Usage

```
/ruview-densepose
/ruview-densepose --once          # one-shot via /console with --once
/ruview-densepose --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/ruview-densepose --stop           # stop the cog on the seed
/ruview-densepose --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"ruview-densepose"}`
- `POST /api/v1/apps/ruview-densepose/start`
- `POST /api/v1/apps/ruview-densepose/stop`
- `POST /api/v1/apps/ruview-densepose/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/ruview-densepose/logs?lines=N`
- `GET  /api/v1/apps/ruview-densepose/manifest`
- `GET  /api/v1/apps/ruview-densepose/config`
- `PUT  /api/v1/apps/ruview-densepose/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Seconds between pose estimations |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/ruview-densepose/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/ruview-densepose/`
