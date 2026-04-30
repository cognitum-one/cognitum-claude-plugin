---
description: Install (if needed) and run the `swarm-witness-federation` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-witness-federation — Witness Chain Federation

Cognitum cog: **Witness Chain Federation**

Share tamper-proof audit trails across seeds

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-witness-federation` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-witness-federation"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-witness-federation/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-witness-federation/logs?lines=5`) and report.

## Usage

```
/swarm-witness-federation
/swarm-witness-federation --once          # one-shot via /console with --once
/swarm-witness-federation --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-witness-federation --stop           # stop the cog on the seed
/swarm-witness-federation --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-witness-federation"}`
- `POST /api/v1/apps/swarm-witness-federation/start`
- `POST /api/v1/apps/swarm-witness-federation/stop`
- `POST /api/v1/apps/swarm-witness-federation/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-witness-federation/logs?lines=N`
- `GET  /api/v1/apps/swarm-witness-federation/manifest`
- `GET  /api/v1/apps/swarm-witness-federation/config`
- `PUT  /api/v1/apps/swarm-witness-federation/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `30` | Seconds between federation rounds |
| `peers` | string | `` | Comma-separated IP addresses of peer seeds in the federation |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-witness-federation/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-witness-federation/`
