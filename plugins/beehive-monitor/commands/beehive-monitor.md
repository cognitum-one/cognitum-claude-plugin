---
description: Install (if needed) and run the `beehive-monitor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /beehive-monitor — Beehive Monitor

Cognitum cog: **Beehive Monitor**

Acoustic hive state classifier. Detects healthy / chaotic / queenless / swarming / robbing via hum-band energy + chaos + piping autocorr

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `beehive-monitor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"beehive-monitor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/beehive-monitor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/beehive-monitor/logs?lines=5`) and report.

## Usage

```
/beehive-monitor
/beehive-monitor --once          # one-shot via /console with --once
/beehive-monitor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/beehive-monitor --stop           # stop the cog on the seed
/beehive-monitor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"beehive-monitor"}`
- `POST /api/v1/apps/beehive-monitor/start`
- `POST /api/v1/apps/beehive-monitor/stop`
- `POST /api/v1/apps/beehive-monitor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/beehive-monitor/logs?lines=N`
- `GET  /api/v1/apps/beehive-monitor/manifest`
- `GET  /api/v1/apps/beehive-monitor/config`
- `PUT  /api/v1/apps/beehive-monitor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Sampling interval |
| `chaos_z` | float | `1.5` | Chaos z-score |
| `robbing_z` | float | `3.0` | Robbing z-score |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/beehive-monitor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/beehive-monitor/`
