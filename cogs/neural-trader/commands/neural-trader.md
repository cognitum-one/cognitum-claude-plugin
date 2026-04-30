---
description: Install (if needed) and run the `neural-trader` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /neural-trader — Neural Trader

Cognitum cog: **Neural Trader**

Spot market patterns and trends from live data

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `neural-trader` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"neural-trader"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/neural-trader/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/neural-trader/logs?lines=5`) and report.

## Usage

```
/neural-trader
/neural-trader --once          # one-shot via /console with --once
/neural-trader --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/neural-trader --stop           # stop the cog on the seed
/neural-trader --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"neural-trader"}`
- `POST /api/v1/apps/neural-trader/start`
- `POST /api/v1/apps/neural-trader/stop`
- `POST /api/v1/apps/neural-trader/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/neural-trader/logs?lines=N`
- `GET  /api/v1/apps/neural-trader/manifest`
- `GET  /api/v1/apps/neural-trader/config`
- `PUT  /api/v1/apps/neural-trader/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `60` | Seconds between market data analysis cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/neural-trader/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/neural-trader/`
