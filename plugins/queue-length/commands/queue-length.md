---
description: Install (if needed) and run the `queue-length` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /queue-length — Queue Length

Cognitum cog: **Queue Length**

Estimates line length and wait time

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `queue-length` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"queue-length"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/queue-length/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/queue-length/logs?lines=5`) and report.

## Usage

```
/queue-length
/queue-length --once          # one-shot via /console with --once
/queue-length --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/queue-length --stop           # stop the cog on the seed
/queue-length --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"queue-length"}`
- `POST /api/v1/apps/queue-length/start`
- `POST /api/v1/apps/queue-length/stop`
- `POST /api/v1/apps/queue-length/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/queue-length/logs?lines=N`
- `GET  /api/v1/apps/queue-length/manifest`
- `GET  /api/v1/apps/queue-length/config`
- `PUT  /api/v1/apps/queue-length/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/queue-length/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/queue-length/`
