---
description: Install (if needed) and run the `sleep-apnea` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /sleep-apnea — Sleep Apnea Detector

Cognitum cog: **Sleep Apnea Detector**

Detects when someone stops breathing during sleep

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `sleep-apnea` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"sleep-apnea"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/sleep-apnea/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/sleep-apnea/logs?lines=5`) and report.

## Usage

```
/sleep-apnea
/sleep-apnea --once          # one-shot via /console with --once
/sleep-apnea --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/sleep-apnea --stop           # stop the cog on the seed
/sleep-apnea --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"sleep-apnea"}`
- `POST /api/v1/apps/sleep-apnea/start`
- `POST /api/v1/apps/sleep-apnea/stop`
- `POST /api/v1/apps/sleep-apnea/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/sleep-apnea/logs?lines=N`
- `GET  /api/v1/apps/sleep-apnea/manifest`
- `GET  /api/v1/apps/sleep-apnea/config`
- `PUT  /api/v1/apps/sleep-apnea/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/sleep-apnea/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/sleep-apnea/`
