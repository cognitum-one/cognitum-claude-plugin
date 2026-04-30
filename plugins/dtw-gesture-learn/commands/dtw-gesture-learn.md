---
description: Install (if needed) and run the `dtw-gesture-learn` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /dtw-gesture-learn — DTW Gesture Learn

Cognitum cog: **DTW Gesture Learn**

Teach custom hand gestures by showing examples

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `dtw-gesture-learn` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"dtw-gesture-learn"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/dtw-gesture-learn/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/dtw-gesture-learn/logs?lines=5`) and report.

## Usage

```
/dtw-gesture-learn
/dtw-gesture-learn --once          # one-shot via /console with --once
/dtw-gesture-learn --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/dtw-gesture-learn --stop           # stop the cog on the seed
/dtw-gesture-learn --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"dtw-gesture-learn"}`
- `POST /api/v1/apps/dtw-gesture-learn/start`
- `POST /api/v1/apps/dtw-gesture-learn/stop`
- `POST /api/v1/apps/dtw-gesture-learn/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/dtw-gesture-learn/logs?lines=N`
- `GET  /api/v1/apps/dtw-gesture-learn/manifest`
- `GET  /api/v1/apps/dtw-gesture-learn/config`
- `PUT  /api/v1/apps/dtw-gesture-learn/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between gesture matching attempts |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/dtw-gesture-learn/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/dtw-gesture-learn/`
