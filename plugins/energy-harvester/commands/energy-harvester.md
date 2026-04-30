---
description: Install (if needed) and run the `energy-harvester` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /energy-harvester — Energy Harvester

Cognitum cog: **Energy Harvester**

Optimize solar and battery for off-grid seed deployment

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `energy-harvester` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"energy-harvester"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/energy-harvester/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/energy-harvester/logs?lines=5`) and report.

## Usage

```
/energy-harvester
/energy-harvester --once          # one-shot via /console with --once
/energy-harvester --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/energy-harvester --stop           # stop the cog on the seed
/energy-harvester --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"energy-harvester"}`
- `POST /api/v1/apps/energy-harvester/start`
- `POST /api/v1/apps/energy-harvester/stop`
- `POST /api/v1/apps/energy-harvester/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/energy-harvester/logs?lines=N`
- `GET  /api/v1/apps/energy-harvester/manifest`
- `GET  /api/v1/apps/energy-harvester/config`
- `PUT  /api/v1/apps/energy-harvester/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/energy-harvester/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/energy-harvester/`
