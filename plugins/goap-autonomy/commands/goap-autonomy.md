---
description: Install (if needed) and run the `goap-autonomy` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /goap-autonomy — GOAP Autonomy

Cognitum cog: **GOAP Autonomy**

Plans and executes goals on its own

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `goap-autonomy` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"goap-autonomy"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/goap-autonomy/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/goap-autonomy/logs?lines=5`) and report.

## Usage

```
/goap-autonomy
/goap-autonomy --once          # one-shot via /console with --once
/goap-autonomy --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/goap-autonomy --stop           # stop the cog on the seed
/goap-autonomy --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"goap-autonomy"}`
- `POST /api/v1/apps/goap-autonomy/start`
- `POST /api/v1/apps/goap-autonomy/stop`
- `POST /api/v1/apps/goap-autonomy/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/goap-autonomy/logs?lines=N`
- `GET  /api/v1/apps/goap-autonomy/manifest`
- `GET  /api/v1/apps/goap-autonomy/config`
- `PUT  /api/v1/apps/goap-autonomy/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between planning cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/goap-autonomy/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/goap-autonomy/`
