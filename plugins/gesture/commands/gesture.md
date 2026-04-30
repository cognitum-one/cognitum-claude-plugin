---
description: Install (if needed) and run the `gesture` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /gesture — Gesture Recognition

Cognitum cog: **Gesture Recognition**

Core gesture recognition building block for cogs

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `gesture` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"gesture"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/gesture/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/gesture/logs?lines=5`) and report.

## Usage

```
/gesture
/gesture --once          # one-shot via /console with --once
/gesture --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/gesture --stop           # stop the cog on the seed
/gesture --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"gesture"}`
- `POST /api/v1/apps/gesture/start`
- `POST /api/v1/apps/gesture/stop`
- `POST /api/v1/apps/gesture/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/gesture/logs?lines=N`
- `GET  /api/v1/apps/gesture/manifest`
- `GET  /api/v1/apps/gesture/config`
- `PUT  /api/v1/apps/gesture/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between gesture checks |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/gesture/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/gesture/`
