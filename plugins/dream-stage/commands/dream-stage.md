---
description: Install (if needed) and run the `dream-stage` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /dream-stage — Dream Stage

Cognitum cog: **Dream Stage**

Tracks your sleep stages — light, deep, and dreaming

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `dream-stage` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"dream-stage"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/dream-stage/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/dream-stage/logs?lines=5`) and report.

## Usage

```
/dream-stage
/dream-stage --once          # one-shot via /console with --once
/dream-stage --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/dream-stage --stop           # stop the cog on the seed
/dream-stage --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"dream-stage"}`
- `POST /api/v1/apps/dream-stage/start`
- `POST /api/v1/apps/dream-stage/stop`
- `POST /api/v1/apps/dream-stage/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/dream-stage/logs?lines=N`
- `GET  /api/v1/apps/dream-stage/manifest`
- `GET  /api/v1/apps/dream-stage/config`
- `PUT  /api/v1/apps/dream-stage/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `60` | Seconds between sleep stage evaluations |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/dream-stage/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/dream-stage/`
