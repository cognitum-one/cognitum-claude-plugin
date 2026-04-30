---
description: Install (if needed) and run the `spiking-tracker` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /spiking-tracker — Spiking Tracker

Cognitum cog: **Spiking Tracker**

Brain-inspired tracker that runs on tiny hardware

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `spiking-tracker` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"spiking-tracker"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/spiking-tracker/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/spiking-tracker/logs?lines=5`) and report.

## Usage

```
/spiking-tracker
/spiking-tracker --once          # one-shot via /console with --once
/spiking-tracker --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/spiking-tracker --stop           # stop the cog on the seed
/spiking-tracker --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"spiking-tracker"}`
- `POST /api/v1/apps/spiking-tracker/start`
- `POST /api/v1/apps/spiking-tracker/stop`
- `POST /api/v1/apps/spiking-tracker/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/spiking-tracker/logs?lines=N`
- `GET  /api/v1/apps/spiking-tracker/manifest`
- `GET  /api/v1/apps/spiking-tracker/config`
- `PUT  /api/v1/apps/spiking-tracker/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/spiking-tracker/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/spiking-tracker/`
