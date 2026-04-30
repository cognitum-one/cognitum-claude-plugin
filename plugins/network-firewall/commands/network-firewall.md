---
description: Install (if needed) and run the `network-firewall` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /network-firewall — Network Firewall

Cognitum cog: **Network Firewall**

Block unauthorized network access per cog

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `network-firewall` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"network-firewall"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/network-firewall/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/network-firewall/logs?lines=5`) and report.

## Usage

```
/network-firewall
/network-firewall --once          # one-shot via /console with --once
/network-firewall --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/network-firewall --stop           # stop the cog on the seed
/network-firewall --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"network-firewall"}`
- `POST /api/v1/apps/network-firewall/start`
- `POST /api/v1/apps/network-firewall/stop`
- `POST /api/v1/apps/network-firewall/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/network-firewall/logs?lines=N`
- `GET  /api/v1/apps/network-firewall/manifest`
- `GET  /api/v1/apps/network-firewall/config`
- `PUT  /api/v1/apps/network-firewall/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between network scans |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/network-firewall/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/network-firewall/`
