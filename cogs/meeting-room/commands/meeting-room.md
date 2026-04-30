---
description: Install (if needed) and run the `meeting-room` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /meeting-room — Meeting Room

Cognitum cog: **Meeting Room**

Shows if a meeting room is free or occupied

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `meeting-room` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"meeting-room"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/meeting-room/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/meeting-room/logs?lines=5`) and report.

## Usage

```
/meeting-room
/meeting-room --once          # one-shot via /console with --once
/meeting-room --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/meeting-room --stop           # stop the cog on the seed
/meeting-room --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"meeting-room"}`
- `POST /api/v1/apps/meeting-room/start`
- `POST /api/v1/apps/meeting-room/stop`
- `POST /api/v1/apps/meeting-room/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/meeting-room/logs?lines=N`
- `GET  /api/v1/apps/meeting-room/manifest`
- `GET  /api/v1/apps/meeting-room/config`
- `PUT  /api/v1/apps/meeting-room/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/meeting-room/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/meeting-room/`
