---
description: Install (if needed) and run the `sparse-recovery` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /sparse-recovery — Sparse Recovery

Cognitum cog: **Sparse Recovery**

Recovers missing signal data from partial readings

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `sparse-recovery` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"sparse-recovery"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/sparse-recovery/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/sparse-recovery/logs?lines=5`) and report.

## Usage

```
/sparse-recovery
/sparse-recovery --once          # one-shot via /console with --once
/sparse-recovery --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/sparse-recovery --stop           # stop the cog on the seed
/sparse-recovery --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"sparse-recovery"}`
- `POST /api/v1/apps/sparse-recovery/start`
- `POST /api/v1/apps/sparse-recovery/stop`
- `POST /api/v1/apps/sparse-recovery/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/sparse-recovery/logs?lines=N`
- `GET  /api/v1/apps/sparse-recovery/manifest`
- `GET  /api/v1/apps/sparse-recovery/config`
- `PUT  /api/v1/apps/sparse-recovery/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/sparse-recovery/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/sparse-recovery/`
