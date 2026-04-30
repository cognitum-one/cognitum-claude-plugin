---
description: Install (if needed) and run the `clean-room` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /clean-room — Clean Room

Cognitum cog: **Clean Room**

Enforces max headcount in controlled environments

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `clean-room` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"clean-room"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/clean-room/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/clean-room/logs?lines=5`) and report.

## Usage

```
/clean-room
/clean-room --once          # one-shot via /console with --once
/clean-room --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/clean-room --stop           # stop the cog on the seed
/clean-room --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"clean-room"}`
- `POST /api/v1/apps/clean-room/start`
- `POST /api/v1/apps/clean-room/stop`
- `POST /api/v1/apps/clean-room/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/clean-room/logs?lines=N`
- `GET  /api/v1/apps/clean-room/manifest`
- `GET  /api/v1/apps/clean-room/config`
- `PUT  /api/v1/apps/clean-room/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `3` | Seconds between measurements |
| `max_occupancy` | integer | `4` | Maximum allowed number of people before alert triggers |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/clean-room/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/clean-room/`
