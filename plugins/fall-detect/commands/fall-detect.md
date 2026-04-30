---
description: Install (if needed) and run the `fall-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /fall-detect — Fall Detection

Cognitum cog: **Fall Detection**

Two-stage impact + stillness fall detector over ambient feature stream (ESP32 motion / mic). Optional ruview-mode for CSI-based pose reinforcement

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `fall-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"fall-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/fall-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/fall-detect/logs?lines=5`) and report.

## Usage

```
/fall-detect
/fall-detect --once          # one-shot via /console with --once
/fall-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/fall-detect --stop           # stop the cog on the seed
/fall-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"fall-detect"}`
- `POST /api/v1/apps/fall-detect/start`
- `POST /api/v1/apps/fall-detect/stop`
- `POST /api/v1/apps/fall-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/fall-detect/logs?lines=N`
- `GET  /api/v1/apps/fall-detect/manifest`
- `GET  /api/v1/apps/fall-detect/config`
- `PUT  /api/v1/apps/fall-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `impact_threshold` | float | `6.0` | Z-score of variance spike that flags an impact event |
| `stillness_window` | integer | `8` | Seconds after impact in which post-fall stillness must occur |
| `cooldown` | integer | `30` | Seconds to suppress further firing after a detected fall |
| `ruview_mode` | boolean | `False` | Reinforce impact stage with WiFi-CSI head-height proxy if present |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/fall-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/fall-detect/`
