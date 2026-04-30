---
description: Install (if needed) and run the `gesture-language` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /gesture-language — Gesture Language

Cognitum cog: **Gesture Language**

Recognizes sign language gestures in real time

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `gesture-language` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"gesture-language"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/gesture-language/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/gesture-language/logs?lines=5`) and report.

## Usage

```
/gesture-language
/gesture-language --once          # one-shot via /console with --once
/gesture-language --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/gesture-language --stop           # stop the cog on the seed
/gesture-language --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"gesture-language"}`
- `POST /api/v1/apps/gesture-language/start`
- `POST /api/v1/apps/gesture-language/stop`
- `POST /api/v1/apps/gesture-language/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/gesture-language/logs?lines=N`
- `GET  /api/v1/apps/gesture-language/manifest`
- `GET  /api/v1/apps/gesture-language/config`
- `PUT  /api/v1/apps/gesture-language/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between gesture matching attempts |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/gesture-language/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/gesture-language/`
