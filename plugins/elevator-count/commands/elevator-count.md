---
description: Install (if needed) and run the `elevator-count` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /elevator-count — Elevator Count

Cognitum cog: **Elevator Count**

Counts how many people are in an elevator

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `elevator-count` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"elevator-count"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/elevator-count/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/elevator-count/logs?lines=5`) and report.

## Usage

```
/elevator-count
/elevator-count --once          # one-shot via /console with --once
/elevator-count --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/elevator-count --stop           # stop the cog on the seed
/elevator-count --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"elevator-count"}`
- `POST /api/v1/apps/elevator-count/start`
- `POST /api/v1/apps/elevator-count/stop`
- `POST /api/v1/apps/elevator-count/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/elevator-count/logs?lines=N`
- `GET  /api/v1/apps/elevator-count/manifest`
- `GET  /api/v1/apps/elevator-count/config`
- `PUT  /api/v1/apps/elevator-count/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/elevator-count/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/elevator-count/`
