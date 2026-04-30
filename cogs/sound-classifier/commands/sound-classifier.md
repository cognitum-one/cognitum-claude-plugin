---
description: Install (if needed) and run the `sound-classifier` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /sound-classifier — Sound Classifier

Cognitum cog: **Sound Classifier**

Identify sounds like glass break, alarm, or baby cry

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `sound-classifier` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"sound-classifier"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/sound-classifier/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/sound-classifier/logs?lines=5`) and report.

## Usage

```
/sound-classifier
/sound-classifier --once          # one-shot via /console with --once
/sound-classifier --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/sound-classifier --stop           # stop the cog on the seed
/sound-classifier --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"sound-classifier"}`
- `POST /api/v1/apps/sound-classifier/start`
- `POST /api/v1/apps/sound-classifier/stop`
- `POST /api/v1/apps/sound-classifier/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/sound-classifier/logs?lines=N`
- `GET  /api/v1/apps/sound-classifier/manifest`
- `GET  /api/v1/apps/sound-classifier/config`
- `PUT  /api/v1/apps/sound-classifier/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between classifications |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/sound-classifier/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/sound-classifier/`
