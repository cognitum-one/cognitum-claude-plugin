---
description: Install (if needed) and run the `predictive-maintenance` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /predictive-maintenance — Predictive Maintenance

Cognitum cog: **Predictive Maintenance**

Vibration harmonic analyzer for rotating equipment. Tracks F1 / 2×F1 / high-order / sideband energy to score degradation severity

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `predictive-maintenance` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"predictive-maintenance"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/predictive-maintenance/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/predictive-maintenance/logs?lines=5`) and report.

## Usage

```
/predictive-maintenance
/predictive-maintenance --once          # one-shot via /console with --once
/predictive-maintenance --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/predictive-maintenance --stop           # stop the cog on the seed
/predictive-maintenance --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"predictive-maintenance"}`
- `POST /api/v1/apps/predictive-maintenance/start`
- `POST /api/v1/apps/predictive-maintenance/stop`
- `POST /api/v1/apps/predictive-maintenance/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/predictive-maintenance/logs?lines=N`
- `GET  /api/v1/apps/predictive-maintenance/manifest`
- `GET  /api/v1/apps/predictive-maintenance/config`
- `PUT  /api/v1/apps/predictive-maintenance/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `baseline_mins` | integer | `5` | Baseline learn period |
| `severity_warn` | float | `0.4` | Severity warn threshold |
| `severity_alarm` | float | `0.7` | Severity alarm threshold |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/predictive-maintenance/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/predictive-maintenance/`
