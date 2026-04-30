---
description: Install (if needed) and run the `federated-learning` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /federated-learning — Federated Learning

Cognitum cog: **Federated Learning**

Train AI across seeds without sharing raw data

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `federated-learning` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"federated-learning"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/federated-learning/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/federated-learning/logs?lines=5`) and report.

## Usage

```
/federated-learning
/federated-learning --once          # one-shot via /console with --once
/federated-learning --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/federated-learning --stop           # stop the cog on the seed
/federated-learning --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"federated-learning"}`
- `POST /api/v1/apps/federated-learning/start`
- `POST /api/v1/apps/federated-learning/stop`
- `POST /api/v1/apps/federated-learning/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/federated-learning/logs?lines=N`
- `GET  /api/v1/apps/federated-learning/manifest`
- `GET  /api/v1/apps/federated-learning/config`
- `PUT  /api/v1/apps/federated-learning/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `30` | Seconds between federation rounds |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/federated-learning/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/federated-learning/`
