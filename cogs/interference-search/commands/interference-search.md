---
description: Install (if needed) and run the `interference-search` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /interference-search — Interference Search

Cognitum cog: **Interference Search**

Searches many possibilities at once for fast answers

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `interference-search` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"interference-search"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/interference-search/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/interference-search/logs?lines=5`) and report.

## Usage

```
/interference-search
/interference-search --once          # one-shot via /console with --once
/interference-search --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/interference-search --stop           # stop the cog on the seed
/interference-search --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"interference-search"}`
- `POST /api/v1/apps/interference-search/start`
- `POST /api/v1/apps/interference-search/stop`
- `POST /api/v1/apps/interference-search/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/interference-search/logs?lines=N`
- `GET  /api/v1/apps/interference-search/manifest`
- `GET  /api/v1/apps/interference-search/config`
- `PUT  /api/v1/apps/interference-search/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/interference-search/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/interference-search/`
