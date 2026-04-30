---
description: Install (if needed) and run the `customer-flow` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /customer-flow — Customer Flow

Cognitum cog: **Customer Flow**

Counts foot traffic in and out of each entrance

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `customer-flow` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"customer-flow"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/customer-flow/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/customer-flow/logs?lines=5`) and report.

## Usage

```
/customer-flow
/customer-flow --once          # one-shot via /console with --once
/customer-flow --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/customer-flow --stop           # stop the cog on the seed
/customer-flow --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"customer-flow"}`
- `POST /api/v1/apps/customer-flow/start`
- `POST /api/v1/apps/customer-flow/stop`
- `POST /api/v1/apps/customer-flow/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/customer-flow/logs?lines=N`
- `GET  /api/v1/apps/customer-flow/manifest`
- `GET  /api/v1/apps/customer-flow/config`
- `PUT  /api/v1/apps/customer-flow/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/customer-flow/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/customer-flow/`
