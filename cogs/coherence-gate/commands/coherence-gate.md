---
description: Install (if needed) and run the `coherence-gate` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /coherence-gate — Coherence Gate

Cognitum cog: **Coherence Gate**

Filters out noisy signals and keeps clean ones

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `coherence-gate` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"coherence-gate"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/coherence-gate/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/coherence-gate/logs?lines=5`) and report.

## Usage

```
/coherence-gate
/coherence-gate --once          # one-shot via /console with --once
/coherence-gate --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/coherence-gate --stop           # stop the cog on the seed
/coherence-gate --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"coherence-gate"}`
- `POST /api/v1/apps/coherence-gate/start`
- `POST /api/v1/apps/coherence-gate/stop`
- `POST /api/v1/apps/coherence-gate/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/coherence-gate/logs?lines=N`
- `GET  /api/v1/apps/coherence-gate/manifest`
- `GET  /api/v1/apps/coherence-gate/config`
- `PUT  /api/v1/apps/coherence-gate/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |
| `threshold` | float | `0.7` | Minimum coherence level to pass signals through the gate |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/coherence-gate/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/coherence-gate/`
