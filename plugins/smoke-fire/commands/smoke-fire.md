---
description: Install (if needed) and run the `smoke-fire` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /smoke-fire — Smoke / Fire Detection

Cognitum cog: **Smoke / Fire Detection**

Multi-signal smoke and fire detector. Fuses acoustic crackle, thermal drift proxy, and optional ruview CSI plume signature. Not a UL-listed replacement for code-required smoke alarms

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `smoke-fire` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"smoke-fire"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/smoke-fire/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/smoke-fire/logs?lines=5`) and report.

## Usage

```
/smoke-fire
/smoke-fire --once          # one-shot via /console with --once
/smoke-fire --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/smoke-fire --stop           # stop the cog on the seed
/smoke-fire --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"smoke-fire"}`
- `POST /api/v1/apps/smoke-fire/start`
- `POST /api/v1/apps/smoke-fire/stop`
- `POST /api/v1/apps/smoke-fire/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/smoke-fire/logs?lines=N`
- `GET  /api/v1/apps/smoke-fire/manifest`
- `GET  /api/v1/apps/smoke-fire/config`
- `PUT  /api/v1/apps/smoke-fire/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `crackle_z` | float | `2.5` | Crackle z-score |
| `thermal_drift_z` | float | `1.5` | Thermal drift z-score |
| `cooldown` | integer | `60` | Cooldown after fire |
| `ruview_mode` | boolean | `False` | Add CSI plume signature as a third evidence stream |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/smoke-fire/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/smoke-fire/`
