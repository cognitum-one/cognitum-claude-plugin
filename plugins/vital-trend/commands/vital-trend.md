---
description: Install (if needed) and run the `vital-trend` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /vital-trend — Vital Trend

Cognitum cog: **Vital Trend**

Tracks breathing and heart rate trends over weeks

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `vital-trend` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"vital-trend"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/vital-trend/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/vital-trend/logs?lines=5`) and report.

## Usage

```
/vital-trend
/vital-trend --once          # one-shot via /console with --once
/vital-trend --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/vital-trend --stop           # stop the cog on the seed
/vital-trend --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"vital-trend"}`
- `POST /api/v1/apps/vital-trend/start`
- `POST /api/v1/apps/vital-trend/stop`
- `POST /api/v1/apps/vital-trend/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/vital-trend/logs?lines=N`
- `GET  /api/v1/apps/vital-trend/manifest`
- `GET  /api/v1/apps/vital-trend/config`
- `PUT  /api/v1/apps/vital-trend/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |
| `breathing_low` | float | `0.1` | Lower bound of bandpass filter for breathing extraction |
| `breathing_high` | float | `0.5` | Upper bound of bandpass filter for breathing extraction |
| `hr_low` | float | `0.8` | Heart rate filter low frequency |
| `hr_high` | float | `2.0` | Heart rate filter high frequency |
| `tachypnea_threshold` | float | `30.0` | Breathing rate above this triggers alert |
| `tachycardia_threshold` | float | `100.0` | Heart rate above this triggers alert |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/vital-trend/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/vital-trend/`
