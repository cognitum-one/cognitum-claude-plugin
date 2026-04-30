---
description: Install (if needed) and run the `optimal-transport` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /optimal-transport — Optimal Transport

Cognitum cog: **Optimal Transport**

Measures motion using shape-aware signal comparison

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `optimal-transport` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"optimal-transport"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/optimal-transport/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/optimal-transport/logs?lines=5`) and report.

## Usage

```
/optimal-transport
/optimal-transport --once          # one-shot via /console with --once
/optimal-transport --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/optimal-transport --stop           # stop the cog on the seed
/optimal-transport --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"optimal-transport"}`
- `POST /api/v1/apps/optimal-transport/start`
- `POST /api/v1/apps/optimal-transport/stop`
- `POST /api/v1/apps/optimal-transport/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/optimal-transport/logs?lines=N`
- `GET  /api/v1/apps/optimal-transport/manifest`
- `GET  /api/v1/apps/optimal-transport/config`
- `PUT  /api/v1/apps/optimal-transport/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/optimal-transport/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/optimal-transport/`
