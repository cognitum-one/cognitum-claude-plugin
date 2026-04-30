---
description: Install (if needed) and run the `emotion-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /emotion-detect — Emotion Detection

Cognitum cog: **Emotion Detection**

Reads stress and calm from body language and breathing

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `emotion-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"emotion-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/emotion-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/emotion-detect/logs?lines=5`) and report.

## Usage

```
/emotion-detect
/emotion-detect --once          # one-shot via /console with --once
/emotion-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/emotion-detect --stop           # stop the cog on the seed
/emotion-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"emotion-detect"}`
- `POST /api/v1/apps/emotion-detect/start`
- `POST /api/v1/apps/emotion-detect/stop`
- `POST /api/v1/apps/emotion-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/emotion-detect/logs?lines=N`
- `GET  /api/v1/apps/emotion-detect/manifest`
- `GET  /api/v1/apps/emotion-detect/config`
- `PUT  /api/v1/apps/emotion-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/emotion-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/emotion-detect/`
