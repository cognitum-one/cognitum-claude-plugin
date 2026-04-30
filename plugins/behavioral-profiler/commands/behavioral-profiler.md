---
description: Install (if needed) and run the `behavioral-profiler` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /behavioral-profiler — Behavioral Profiler

Cognitum cog: **Behavioral Profiler**

Learns normal behavior and flags anything unusual

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `behavioral-profiler` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"behavioral-profiler"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/behavioral-profiler/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/behavioral-profiler/logs?lines=5`) and report.

## Usage

```
/behavioral-profiler
/behavioral-profiler --once          # one-shot via /console with --once
/behavioral-profiler --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/behavioral-profiler --stop           # stop the cog on the seed
/behavioral-profiler --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"behavioral-profiler"}`
- `POST /api/v1/apps/behavioral-profiler/start`
- `POST /api/v1/apps/behavioral-profiler/stop`
- `POST /api/v1/apps/behavioral-profiler/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/behavioral-profiler/logs?lines=N`
- `GET  /api/v1/apps/behavioral-profiler/manifest`
- `GET  /api/v1/apps/behavioral-profiler/config`
- `PUT  /api/v1/apps/behavioral-profiler/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/behavioral-profiler/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/behavioral-profiler/`
