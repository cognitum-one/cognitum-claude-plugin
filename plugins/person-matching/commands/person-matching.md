---
description: Install (if needed) and run the `person-matching` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /person-matching — Person Matching

Cognitum cog: **Person Matching**

Tells apart multiple people in the same room

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `person-matching` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"person-matching"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/person-matching/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/person-matching/logs?lines=5`) and report.

## Usage

```
/person-matching
/person-matching --once          # one-shot via /console with --once
/person-matching --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/person-matching --stop           # stop the cog on the seed
/person-matching --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"person-matching"}`
- `POST /api/v1/apps/person-matching/start`
- `POST /api/v1/apps/person-matching/stop`
- `POST /api/v1/apps/person-matching/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/person-matching/logs?lines=N`
- `GET  /api/v1/apps/person-matching/manifest`
- `GET  /api/v1/apps/person-matching/config`
- `PUT  /api/v1/apps/person-matching/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |
| `max_people` | integer | `5` | Maximum number of distinct people to track simultaneously |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/person-matching/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/person-matching/`
