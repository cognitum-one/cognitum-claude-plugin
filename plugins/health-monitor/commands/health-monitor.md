---
description: Install (if needed) and run the `health-monitor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /health-monitor — Health Monitor

Cognitum cog: **Health Monitor**

Contactless heart rate, breathing, sleep, and fall alerts

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `health-monitor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"health-monitor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/health-monitor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/health-monitor/logs?lines=5`) and report.

## Usage

```
/health-monitor
/health-monitor --once          # one-shot via /console with --once
/health-monitor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/health-monitor --stop           # stop the cog on the seed
/health-monitor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"health-monitor"}`
- `POST /api/v1/apps/health-monitor/start`
- `POST /api/v1/apps/health-monitor/stop`
- `POST /api/v1/apps/health-monitor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/health-monitor/logs?lines=N`
- `GET  /api/v1/apps/health-monitor/manifest`
- `GET  /api/v1/apps/health-monitor/config`
- `PUT  /api/v1/apps/health-monitor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/health-monitor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/health-monitor/`
