---
description: Install (if needed) and run the `energy-audit` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /energy-audit — Energy Audit

Cognitum cog: **Energy Audit**

Learns your schedule and cuts wasted energy

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `energy-audit` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"energy-audit"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/energy-audit/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/energy-audit/logs?lines=5`) and report.

## Usage

```
/energy-audit
/energy-audit --once          # one-shot via /console with --once
/energy-audit --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/energy-audit --stop           # stop the cog on the seed
/energy-audit --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"energy-audit"}`
- `POST /api/v1/apps/energy-audit/start`
- `POST /api/v1/apps/energy-audit/stop`
- `POST /api/v1/apps/energy-audit/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/energy-audit/logs?lines=N`
- `GET  /api/v1/apps/energy-audit/manifest`
- `GET  /api/v1/apps/energy-audit/config`
- `PUT  /api/v1/apps/energy-audit/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `60` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/energy-audit/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/energy-audit/`
