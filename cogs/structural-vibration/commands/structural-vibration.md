---
description: Install (if needed) and run the `structural-vibration` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /structural-vibration — Structural Vibration

Cognitum cog: **Structural Vibration**

Detects dangerous vibrations in buildings or machines

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `structural-vibration` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"structural-vibration"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/structural-vibration/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/structural-vibration/logs?lines=5`) and report.

## Usage

```
/structural-vibration
/structural-vibration --once          # one-shot via /console with --once
/structural-vibration --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/structural-vibration --stop           # stop the cog on the seed
/structural-vibration --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"structural-vibration"}`
- `POST /api/v1/apps/structural-vibration/start`
- `POST /api/v1/apps/structural-vibration/stop`
- `POST /api/v1/apps/structural-vibration/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/structural-vibration/logs?lines=N`
- `GET  /api/v1/apps/structural-vibration/manifest`
- `GET  /api/v1/apps/structural-vibration/config`
- `PUT  /api/v1/apps/structural-vibration/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Seconds between measurements |
| `threshold` | float | `50.0` | RMS energy level above which vibration is considered dangerous |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/structural-vibration/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/structural-vibration/`
