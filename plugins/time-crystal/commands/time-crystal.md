---
description: Install (if needed) and run the `time-crystal` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /time-crystal — Time Crystal

Cognitum cog: **Time Crystal**

Experiments with repeating time-pattern symmetry

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `time-crystal` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"time-crystal"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/time-crystal/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/time-crystal/logs?lines=5`) and report.

## Usage

```
/time-crystal
/time-crystal --once          # one-shot via /console with --once
/time-crystal --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/time-crystal --stop           # stop the cog on the seed
/time-crystal --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"time-crystal"}`
- `POST /api/v1/apps/time-crystal/start`
- `POST /api/v1/apps/time-crystal/stop`
- `POST /api/v1/apps/time-crystal/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/time-crystal/logs?lines=N`
- `GET  /api/v1/apps/time-crystal/manifest`
- `GET  /api/v1/apps/time-crystal/config`
- `PUT  /api/v1/apps/time-crystal/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/time-crystal/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/time-crystal/`
