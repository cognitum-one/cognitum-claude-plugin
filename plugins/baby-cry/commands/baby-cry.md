---
description: Install (if needed) and run the `baby-cry` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /baby-cry — Baby Cry Detection

Cognitum cog: **Baby Cry Detection**

Sustained mid-band energy detector for nursery / infant monitoring. Audio-only, no camera

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `baby-cry` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"baby-cry"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/baby-cry/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/baby-cry/logs?lines=5`) and report.

## Usage

```
/baby-cry
/baby-cry --once          # one-shot via /console with --once
/baby-cry --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/baby-cry --stop           # stop the cog on the seed
/baby-cry --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"baby-cry"}`
- `POST /api/v1/apps/baby-cry/start`
- `POST /api/v1/apps/baby-cry/stop`
- `POST /api/v1/apps/baby-cry/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/baby-cry/logs?lines=N`
- `GET  /api/v1/apps/baby-cry/manifest`
- `GET  /api/v1/apps/baby-cry/config`
- `PUT  /api/v1/apps/baby-cry/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `cry_z` | float | `2.5` | Mid-band z-score threshold above baseline to count as elevated |
| `cry_min_secs` | integer | `2` | Continuous elevated frames required to fire |
| `cooldown` | integer | `15` | Cooldown after fire |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/baby-cry/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/baby-cry/`
