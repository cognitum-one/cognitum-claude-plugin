---
description: Install (if needed) and run the `seizure-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /seizure-detect — Seizure Detection

Cognitum cog: **Seizure Detection**

Recognizes seizures and sends immediate alerts

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `seizure-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"seizure-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/seizure-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/seizure-detect/logs?lines=5`) and report.

## Usage

```
/seizure-detect
/seizure-detect --once          # one-shot via /console with --once
/seizure-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/seizure-detect --stop           # stop the cog on the seed
/seizure-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"seizure-detect"}`
- `POST /api/v1/apps/seizure-detect/start`
- `POST /api/v1/apps/seizure-detect/stop`
- `POST /api/v1/apps/seizure-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/seizure-detect/logs?lines=N`
- `GET  /api/v1/apps/seizure-detect/manifest`
- `GET  /api/v1/apps/seizure-detect/config`
- `PUT  /api/v1/apps/seizure-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `3` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/seizure-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/seizure-detect/`
