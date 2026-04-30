---
description: Install (if needed) and run the `snore-monitor` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /snore-monitor — Snore Monitor

Cognitum cog: **Snore Monitor**

Periodic low-band energy tracker for sleep-quality / apnea-risk trending. Companion to sleep-apnea cog

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `snore-monitor` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"snore-monitor"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/snore-monitor/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/snore-monitor/logs?lines=5`) and report.

## Usage

```
/snore-monitor
/snore-monitor --once          # one-shot via /console with --once
/snore-monitor --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/snore-monitor --stop           # stop the cog on the seed
/snore-monitor --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"snore-monitor"}`
- `POST /api/v1/apps/snore-monitor/start`
- `POST /api/v1/apps/snore-monitor/stop`
- `POST /api/v1/apps/snore-monitor/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/snore-monitor/logs?lines=N`
- `GET  /api/v1/apps/snore-monitor/manifest`
- `GET  /api/v1/apps/snore-monitor/config`
- `PUT  /api/v1/apps/snore-monitor/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `burst_z` | float | `2.0` | Burst z-score |
| `minimum_rate_hz` | float | `1.5` | Lower bound of expected snore repetition rate (Hz) |
| `maximum_rate_hz` | float | `4.0` | Upper bound of expected snore repetition rate (Hz) |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/snore-monitor/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/snore-monitor/`
