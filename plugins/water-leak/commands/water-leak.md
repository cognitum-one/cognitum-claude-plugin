---
description: Install (if needed) and run the `water-leak` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /water-leak — Water Leak Detection

Cognitum cog: **Water Leak Detection**

Persistent low-amplitude hiss + periodic drip acoustic detector with multi-minute persistence gate. Two-stage likely → confirmed

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `water-leak` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"water-leak"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/water-leak/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/water-leak/logs?lines=5`) and report.

## Usage

```
/water-leak
/water-leak --once          # one-shot via /console with --once
/water-leak --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/water-leak --stop           # stop the cog on the seed
/water-leak --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"water-leak"}`
- `POST /api/v1/apps/water-leak/start`
- `POST /api/v1/apps/water-leak/stop`
- `POST /api/v1/apps/water-leak/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/water-leak/logs?lines=N`
- `GET  /api/v1/apps/water-leak/manifest`
- `GET  /api/v1/apps/water-leak/config`
- `PUT  /api/v1/apps/water-leak/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `2` | Sampling interval |
| `hiss_z` | float | `1.5` | Hiss z-score |
| `persistence` | integer | `300` | Seconds the signature must hold to fire LEAK_CONFIRMED |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/water-leak/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/water-leak/`
