---
description: Install (if needed) and run the `glass-break` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /glass-break — Glass-Break Detection

Cognitum cog: **Glass-Break Detection**

Two-phase bang + shatter acoustic detector. Distinguishes glass break from ordinary impulse noise

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `glass-break` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"glass-break"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/glass-break/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/glass-break/logs?lines=5`) and report.

## Usage

```
/glass-break
/glass-break --once          # one-shot via /console with --once
/glass-break --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/glass-break --stop           # stop the cog on the seed
/glass-break --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"glass-break"}`
- `POST /api/v1/apps/glass-break/start`
- `POST /api/v1/apps/glass-break/stop`
- `POST /api/v1/apps/glass-break/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/glass-break/logs?lines=N`
- `GET  /api/v1/apps/glass-break/manifest`
- `GET  /api/v1/apps/glass-break/config`
- `PUT  /api/v1/apps/glass-break/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `bang_z` | float | `5.0` | Bang z-score |
| `shatter_window_ms` | integer | `500` | Milliseconds after bang within which shatter must be observed |
| `cooldown` | integer | `30` | Cooldown after fire |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/glass-break/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/glass-break/`
