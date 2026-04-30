---
description: Install (if needed) and run the `psycho-symbolic` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /psycho-symbolic — Psycho-Symbolic

Cognitum cog: **Psycho-Symbolic**

Reasons over knowledge graphs with multiple styles

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `psycho-symbolic` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"psycho-symbolic"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/psycho-symbolic/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/psycho-symbolic/logs?lines=5`) and report.

## Usage

```
/psycho-symbolic
/psycho-symbolic --once          # one-shot via /console with --once
/psycho-symbolic --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/psycho-symbolic --stop           # stop the cog on the seed
/psycho-symbolic --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"psycho-symbolic"}`
- `POST /api/v1/apps/psycho-symbolic/start`
- `POST /api/v1/apps/psycho-symbolic/stop`
- `POST /api/v1/apps/psycho-symbolic/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/psycho-symbolic/logs?lines=N`
- `GET  /api/v1/apps/psycho-symbolic/manifest`
- `GET  /api/v1/apps/psycho-symbolic/config`
- `PUT  /api/v1/apps/psycho-symbolic/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between reasoning cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/psycho-symbolic/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/psycho-symbolic/`
