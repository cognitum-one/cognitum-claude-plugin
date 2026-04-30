---
description: Install (if needed) and run the `swarm-consensus` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-consensus — Swarm Consensus

Cognitum cog: **Swarm Consensus**

Seeds vote before making critical changes together

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-consensus` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-consensus"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-consensus/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-consensus/logs?lines=5`) and report.

## Usage

```
/swarm-consensus
/swarm-consensus --once          # one-shot via /console with --once
/swarm-consensus --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-consensus --stop           # stop the cog on the seed
/swarm-consensus --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-consensus"}`
- `POST /api/v1/apps/swarm-consensus/start`
- `POST /api/v1/apps/swarm-consensus/stop`
- `POST /api/v1/apps/swarm-consensus/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-consensus/logs?lines=N`
- `GET  /api/v1/apps/swarm-consensus/manifest`
- `GET  /api/v1/apps/swarm-consensus/config`
- `PUT  /api/v1/apps/swarm-consensus/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `60` | Seconds between consensus rounds |
| `peers` | string | `` | Comma-separated IP addresses of peer seeds to vote with |
| `propose` | string | `` | Action to propose for voting (e.g. enable-cog:presence) |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-consensus/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-consensus/`
