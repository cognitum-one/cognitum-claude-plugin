---
description: Install (if needed) and run the `cardiac-arrhythmia` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /cardiac-arrhythmia — Cardiac Arrhythmia

Cognitum cog: **Cardiac Arrhythmia**

Spots irregular heartbeats and abnormal heart rhythms

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `cardiac-arrhythmia` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"cardiac-arrhythmia"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/cardiac-arrhythmia/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/cardiac-arrhythmia/logs?lines=5`) and report.

## Usage

```
/cardiac-arrhythmia
/cardiac-arrhythmia --once          # one-shot via /console with --once
/cardiac-arrhythmia --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/cardiac-arrhythmia --stop           # stop the cog on the seed
/cardiac-arrhythmia --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"cardiac-arrhythmia"}`
- `POST /api/v1/apps/cardiac-arrhythmia/start`
- `POST /api/v1/apps/cardiac-arrhythmia/stop`
- `POST /api/v1/apps/cardiac-arrhythmia/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/cardiac-arrhythmia/logs?lines=N`
- `GET  /api/v1/apps/cardiac-arrhythmia/manifest`
- `GET  /api/v1/apps/cardiac-arrhythmia/config`
- `PUT  /api/v1/apps/cardiac-arrhythmia/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/cardiac-arrhythmia/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/cardiac-arrhythmia/`
