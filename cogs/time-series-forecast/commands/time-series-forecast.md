---
description: Install (if needed) and run the `time-series-forecast` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /time-series-forecast — Time Series Forecast

Cognitum cog: **Time Series Forecast**

Predict sensor trends using historical patterns

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `time-series-forecast` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"time-series-forecast"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/time-series-forecast/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/time-series-forecast/logs?lines=5`) and report.

## Usage

```
/time-series-forecast
/time-series-forecast --once          # one-shot via /console with --once
/time-series-forecast --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/time-series-forecast --stop           # stop the cog on the seed
/time-series-forecast --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"time-series-forecast"}`
- `POST /api/v1/apps/time-series-forecast/start`
- `POST /api/v1/apps/time-series-forecast/stop`
- `POST /api/v1/apps/time-series-forecast/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/time-series-forecast/logs?lines=N`
- `GET  /api/v1/apps/time-series-forecast/manifest`
- `GET  /api/v1/apps/time-series-forecast/config`
- `PUT  /api/v1/apps/time-series-forecast/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between forecast updates |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/time-series-forecast/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/time-series-forecast/`
