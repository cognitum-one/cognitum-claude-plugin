---
description: Install (if needed) and run the `intrusion-detect-ml` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /intrusion-detect-ml — ML Intrusion Detection

Cognitum cog: **ML Intrusion Detection**

Detect network attacks using machine learning

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `intrusion-detect-ml` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"intrusion-detect-ml"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/intrusion-detect-ml/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/intrusion-detect-ml/logs?lines=5`) and report.

## Usage

```
/intrusion-detect-ml
/intrusion-detect-ml --once          # one-shot via /console with --once
/intrusion-detect-ml --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/intrusion-detect-ml --stop           # stop the cog on the seed
/intrusion-detect-ml --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"intrusion-detect-ml"}`
- `POST /api/v1/apps/intrusion-detect-ml/start`
- `POST /api/v1/apps/intrusion-detect-ml/stop`
- `POST /api/v1/apps/intrusion-detect-ml/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/intrusion-detect-ml/logs?lines=N`
- `GET  /api/v1/apps/intrusion-detect-ml/manifest`
- `GET  /api/v1/apps/intrusion-detect-ml/config`
- `PUT  /api/v1/apps/intrusion-detect-ml/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `3` | Seconds between intrusion checks |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/intrusion-detect-ml/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/intrusion-detect-ml/`
