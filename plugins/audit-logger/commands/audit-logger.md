---
description: Install (if needed) and run the `audit-logger` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /audit-logger — Audit Trail Logger

Cognitum cog: **Audit Trail Logger**

Record every action for compliance — tamper-proof log

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `audit-logger` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"audit-logger"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/audit-logger/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/audit-logger/logs?lines=5`) and report.

## Usage

```
/audit-logger
/audit-logger --once          # one-shot via /console with --once
/audit-logger --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/audit-logger --stop           # stop the cog on the seed
/audit-logger --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"audit-logger"}`
- `POST /api/v1/apps/audit-logger/start`
- `POST /api/v1/apps/audit-logger/stop`
- `POST /api/v1/apps/audit-logger/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/audit-logger/logs?lines=N`
- `GET  /api/v1/apps/audit-logger/manifest`
- `GET  /api/v1/apps/audit-logger/config`
- `PUT  /api/v1/apps/audit-logger/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between audit checks |
| `cloud` | string | `` | HTTP endpoint to forward audit events to (e.g. https://audit.example.com/ingest) |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/audit-logger/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/audit-logger/`
