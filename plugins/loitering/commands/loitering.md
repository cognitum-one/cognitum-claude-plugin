---
description: Install (if needed) and run the `loitering` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /loitering — Loitering Detection

Cognitum cog: **Loitering Detection**

Alerts when someone lingers too long in one spot

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `loitering` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"loitering"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/loitering/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/loitering/logs?lines=5`) and report.

## Usage

```
/loitering
/loitering --once          # one-shot via /console with --once
/loitering --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/loitering --stop           # stop the cog on the seed
/loitering --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"loitering"}`
- `POST /api/v1/apps/loitering/start`
- `POST /api/v1/apps/loitering/stop`
- `POST /api/v1/apps/loitering/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/loitering/logs?lines=N`
- `GET  /api/v1/apps/loitering/manifest`
- `GET  /api/v1/apps/loitering/config`
- `PUT  /api/v1/apps/loitering/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |
| `loiter_time` | integer | `120` | Seconds of stationary presence before triggering an alert |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/loitering/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/loitering/`
