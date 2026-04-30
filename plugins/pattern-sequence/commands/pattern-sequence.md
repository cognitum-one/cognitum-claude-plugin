---
description: Install (if needed) and run the `pattern-sequence` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /pattern-sequence — Pattern Sequence

Cognitum cog: **Pattern Sequence**

Detects daily routines and repeated habits

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `pattern-sequence` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"pattern-sequence"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/pattern-sequence/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/pattern-sequence/logs?lines=5`) and report.

## Usage

```
/pattern-sequence
/pattern-sequence --once          # one-shot via /console with --once
/pattern-sequence --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/pattern-sequence --stop           # stop the cog on the seed
/pattern-sequence --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"pattern-sequence"}`
- `POST /api/v1/apps/pattern-sequence/start`
- `POST /api/v1/apps/pattern-sequence/stop`
- `POST /api/v1/apps/pattern-sequence/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/pattern-sequence/logs?lines=N`
- `GET  /api/v1/apps/pattern-sequence/manifest`
- `GET  /api/v1/apps/pattern-sequence/config`
- `PUT  /api/v1/apps/pattern-sequence/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `60` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/pattern-sequence/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/pattern-sequence/`
