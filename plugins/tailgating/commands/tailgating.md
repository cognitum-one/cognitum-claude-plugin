---
description: Install (if needed) and run the `tailgating` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /tailgating — Tailgating Detection

Cognitum cog: **Tailgating Detection**

Catches when someone sneaks in behind a badge holder

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `tailgating` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"tailgating"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/tailgating/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/tailgating/logs?lines=5`) and report.

## Usage

```
/tailgating
/tailgating --once          # one-shot via /console with --once
/tailgating --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/tailgating --stop           # stop the cog on the seed
/tailgating --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"tailgating"}`
- `POST /api/v1/apps/tailgating/start`
- `POST /api/v1/apps/tailgating/stop`
- `POST /api/v1/apps/tailgating/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/tailgating/logs?lines=N`
- `GET  /api/v1/apps/tailgating/manifest`
- `GET  /api/v1/apps/tailgating/config`
- `PUT  /api/v1/apps/tailgating/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/tailgating/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/tailgating/`
