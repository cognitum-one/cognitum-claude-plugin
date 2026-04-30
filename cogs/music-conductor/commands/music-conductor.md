---
description: Install (if needed) and run the `music-conductor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /music-conductor — Music Conductor

Cognitum cog: **Music Conductor**

Reads a conductor's gestures for tempo and dynamics

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `music-conductor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"music-conductor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/music-conductor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/music-conductor/logs?lines=5`) and report.

## Usage

```
/music-conductor
/music-conductor --once          # one-shot via /console with --once
/music-conductor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/music-conductor --stop           # stop the cog on the seed
/music-conductor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"music-conductor"}`
- `POST /api/v1/apps/music-conductor/start`
- `POST /api/v1/apps/music-conductor/stop`
- `POST /api/v1/apps/music-conductor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/music-conductor/logs?lines=N`
- `GET  /api/v1/apps/music-conductor/manifest`
- `GET  /api/v1/apps/music-conductor/config`
- `PUT  /api/v1/apps/music-conductor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |
| `sample_rate` | float | `100.0` | Expected sensor sampling rate for tempo calculation |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/music-conductor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/music-conductor/`
