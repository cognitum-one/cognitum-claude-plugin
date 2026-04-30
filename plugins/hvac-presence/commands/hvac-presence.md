---
description: Install (if needed) and run the `hvac-presence` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /hvac-presence — HVAC Presence

Cognitum cog: **HVAC Presence**

Turns heating and cooling on when you arrive

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `hvac-presence` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"hvac-presence"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/hvac-presence/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/hvac-presence/logs?lines=5`) and report.

## Usage

```
/hvac-presence
/hvac-presence --once          # one-shot via /console with --once
/hvac-presence --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/hvac-presence --stop           # stop the cog on the seed
/hvac-presence --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"hvac-presence"}`
- `POST /api/v1/apps/hvac-presence/start`
- `POST /api/v1/apps/hvac-presence/stop`
- `POST /api/v1/apps/hvac-presence/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/hvac-presence/logs?lines=N`
- `GET  /api/v1/apps/hvac-presence/manifest`
- `GET  /api/v1/apps/hvac-presence/config`
- `PUT  /api/v1/apps/hvac-presence/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/hvac-presence/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/hvac-presence/`
