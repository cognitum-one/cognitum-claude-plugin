---
description: Install (if needed) and run the `fleet-auth` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /fleet-auth — Fleet Authentication

Cognitum cog: **Fleet Authentication**

Manage device certificates and access across all seeds

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `fleet-auth` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"fleet-auth"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/fleet-auth/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/fleet-auth/logs?lines=5`) and report.

## Usage

```
/fleet-auth
/fleet-auth --once          # one-shot via /console with --once
/fleet-auth --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/fleet-auth --stop           # stop the cog on the seed
/fleet-auth --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"fleet-auth"}`
- `POST /api/v1/apps/fleet-auth/start`
- `POST /api/v1/apps/fleet-auth/stop`
- `POST /api/v1/apps/fleet-auth/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/fleet-auth/logs?lines=N`
- `GET  /api/v1/apps/fleet-auth/manifest`
- `GET  /api/v1/apps/fleet-auth/config`
- `PUT  /api/v1/apps/fleet-auth/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `30` | Seconds between authentication verification rounds |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/fleet-auth/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/fleet-auth/`
