---
description: Install (if needed) and run the `temporal-compress` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /temporal-compress — Temporal Compress

Cognitum cog: **Temporal Compress**

Shrinks old data to save memory without losing meaning

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `temporal-compress` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"temporal-compress"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/temporal-compress/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/temporal-compress/logs?lines=5`) and report.

## Usage

```
/temporal-compress
/temporal-compress --once          # one-shot via /console with --once
/temporal-compress --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/temporal-compress --stop           # stop the cog on the seed
/temporal-compress --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"temporal-compress"}`
- `POST /api/v1/apps/temporal-compress/start`
- `POST /api/v1/apps/temporal-compress/stop`
- `POST /api/v1/apps/temporal-compress/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/temporal-compress/logs?lines=N`
- `GET  /api/v1/apps/temporal-compress/manifest`
- `GET  /api/v1/apps/temporal-compress/config`
- `PUT  /api/v1/apps/temporal-compress/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between compression cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/temporal-compress/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/temporal-compress/`
