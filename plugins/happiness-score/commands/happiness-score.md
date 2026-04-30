---
description: Install (if needed) and run the `happiness-score` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /happiness-score — Happiness Score

Cognitum cog: **Happiness Score**

Estimates well-being from movement and mood signals

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `happiness-score` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"happiness-score"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/happiness-score/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/happiness-score/logs?lines=5`) and report.

## Usage

```
/happiness-score
/happiness-score --once          # one-shot via /console with --once
/happiness-score --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/happiness-score --stop           # stop the cog on the seed
/happiness-score --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"happiness-score"}`
- `POST /api/v1/apps/happiness-score/start`
- `POST /api/v1/apps/happiness-score/stop`
- `POST /api/v1/apps/happiness-score/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/happiness-score/logs?lines=N`
- `GET  /api/v1/apps/happiness-score/manifest`
- `GET  /api/v1/apps/happiness-score/config`
- `PUT  /api/v1/apps/happiness-score/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/happiness-score/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/happiness-score/`
