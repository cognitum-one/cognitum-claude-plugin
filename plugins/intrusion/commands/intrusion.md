---
description: Install (if needed) and run the `intrusion` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /intrusion — Intrusion Detection

Cognitum cog: **Intrusion Detection**

Alerts when an unauthorized person enters a room

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `intrusion` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"intrusion"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/intrusion/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/intrusion/logs?lines=5`) and report.

## Usage

```
/intrusion
/intrusion --once          # one-shot via /console with --once
/intrusion --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/intrusion --stop           # stop the cog on the seed
/intrusion --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"intrusion"}`
- `POST /api/v1/apps/intrusion/start`
- `POST /api/v1/apps/intrusion/stop`
- `POST /api/v1/apps/intrusion/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/intrusion/logs?lines=N`
- `GET  /api/v1/apps/intrusion/manifest`
- `GET  /api/v1/apps/intrusion/config`
- `PUT  /api/v1/apps/intrusion/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `3` | Scan interval |
| `arm_after` | integer | `60` | Seconds of baseline learning before arming the detector |
| `detection_threshold` | float | `3.0` | Combined z-score threshold to trigger intrusion alert |
| `trigger_count` | integer | `2` | Number of consecutive detections before alarm |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/intrusion/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/intrusion/`
