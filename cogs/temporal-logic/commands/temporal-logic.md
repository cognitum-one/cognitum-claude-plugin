---
description: Install (if needed) and run the `temporal-logic` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /temporal-logic — Temporal Logic Guard

Cognitum cog: **Temporal Logic Guard**

Enforces safety rules on live event streams

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `temporal-logic` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"temporal-logic"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/temporal-logic/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/temporal-logic/logs?lines=5`) and report.

## Usage

```
/temporal-logic
/temporal-logic --once          # one-shot via /console with --once
/temporal-logic --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/temporal-logic --stop           # stop the cog on the seed
/temporal-logic --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"temporal-logic"}`
- `POST /api/v1/apps/temporal-logic/start`
- `POST /api/v1/apps/temporal-logic/stop`
- `POST /api/v1/apps/temporal-logic/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/temporal-logic/logs?lines=N`
- `GET  /api/v1/apps/temporal-logic/manifest`
- `GET  /api/v1/apps/temporal-logic/config`
- `PUT  /api/v1/apps/temporal-logic/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between rule evaluation cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/temporal-logic/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/temporal-logic/`
