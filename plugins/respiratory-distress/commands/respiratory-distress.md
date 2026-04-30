---
description: Install (if needed) and run the `respiratory-distress` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /respiratory-distress — Respiratory Distress

Cognitum cog: **Respiratory Distress**

Alerts when breathing becomes labored or dangerously fast

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `respiratory-distress` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"respiratory-distress"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/respiratory-distress/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/respiratory-distress/logs?lines=5`) and report.

## Usage

```
/respiratory-distress
/respiratory-distress --once          # one-shot via /console with --once
/respiratory-distress --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/respiratory-distress --stop           # stop the cog on the seed
/respiratory-distress --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"respiratory-distress"}`
- `POST /api/v1/apps/respiratory-distress/start`
- `POST /api/v1/apps/respiratory-distress/stop`
- `POST /api/v1/apps/respiratory-distress/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/respiratory-distress/logs?lines=N`
- `GET  /api/v1/apps/respiratory-distress/manifest`
- `GET  /api/v1/apps/respiratory-distress/config`
- `PUT  /api/v1/apps/respiratory-distress/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/respiratory-distress/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/respiratory-distress/`
