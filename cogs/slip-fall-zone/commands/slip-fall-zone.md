---
description: Install (if needed) and run the `slip-fall-zone` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /slip-fall-zone — Slip / Wet-Floor Zone

Cognitum cog: **Slip / Wet-Floor Zone**

Pre-fall risk detector. Fires when motion-variance drop, splash audio, and optional cautious-gait CSI all signal elevated slip risk

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `slip-fall-zone` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"slip-fall-zone"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/slip-fall-zone/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/slip-fall-zone/logs?lines=5`) and report.

## Usage

```
/slip-fall-zone
/slip-fall-zone --once          # one-shot via /console with --once
/slip-fall-zone --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/slip-fall-zone --stop           # stop the cog on the seed
/slip-fall-zone --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"slip-fall-zone"}`
- `POST /api/v1/apps/slip-fall-zone/start`
- `POST /api/v1/apps/slip-fall-zone/stop`
- `POST /api/v1/apps/slip-fall-zone/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/slip-fall-zone/logs?lines=N`
- `GET  /api/v1/apps/slip-fall-zone/manifest`
- `GET  /api/v1/apps/slip-fall-zone/config`
- `PUT  /api/v1/apps/slip-fall-zone/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `motion_drop_z` | float | `1.5` | Motion drop z-score |
| `splash_z` | float | `3.0` | Splash z-score |
| `cooldown` | integer | `600` | Cooldown after fire |
| `ruview_mode` | boolean | `False` | Add cautious-gait CSI score to risk fusion |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/slip-fall-zone/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/slip-fall-zone/`
