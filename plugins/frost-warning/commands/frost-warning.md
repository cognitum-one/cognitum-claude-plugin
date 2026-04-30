---
description: Install (if needed) and run the `frost-warning` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /frost-warning — Frost Warning

Cognitum cog: **Frost Warning**

Predicts frost 6 hours ahead via temperature trend + dewpoint-depression gate. Field/orchard agriculture

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `frost-warning` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"frost-warning"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/frost-warning/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/frost-warning/logs?lines=5`) and report.

## Usage

```
/frost-warning
/frost-warning --once          # one-shot via /console with --once
/frost-warning --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/frost-warning --stop           # stop the cog on the seed
/frost-warning --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"frost-warning"}`
- `POST /api/v1/apps/frost-warning/start`
- `POST /api/v1/apps/frost-warning/stop`
- `POST /api/v1/apps/frost-warning/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/frost-warning/logs?lines=N`
- `GET  /api/v1/apps/frost-warning/manifest`
- `GET  /api/v1/apps/frost-warning/config`
- `PUT  /api/v1/apps/frost-warning/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Sampling interval |
| `frost_threshold` | float | `2.0` | Frost threshold (Celsius) |
| `projection_hours` | integer | `6` | Projection horizon (hours) |
| `dewpoint_min_depression` | float | `3.0` | Minimum (temp - dewpoint) below which frost-likely fires |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/frost-warning/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/frost-warning/`
