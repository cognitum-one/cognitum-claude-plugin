---
description: Install (if needed) and run the `livestock-monitor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /livestock-monitor — Livestock Monitor

Cognitum cog: **Livestock Monitor**

Monitors animals for distress, escape, or illness

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `livestock-monitor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"livestock-monitor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/livestock-monitor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/livestock-monitor/logs?lines=5`) and report.

## Usage

```
/livestock-monitor
/livestock-monitor --once          # one-shot via /console with --once
/livestock-monitor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/livestock-monitor --stop           # stop the cog on the seed
/livestock-monitor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"livestock-monitor"}`
- `POST /api/v1/apps/livestock-monitor/start`
- `POST /api/v1/apps/livestock-monitor/stop`
- `POST /api/v1/apps/livestock-monitor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/livestock-monitor/logs?lines=N`
- `GET  /api/v1/apps/livestock-monitor/manifest`
- `GET  /api/v1/apps/livestock-monitor/config`
- `PUT  /api/v1/apps/livestock-monitor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/livestock-monitor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/livestock-monitor/`
