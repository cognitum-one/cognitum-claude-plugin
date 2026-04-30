---
description: Install (if needed) and run the `ewc-lifelong` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /ewc-lifelong — EWC Lifelong

Cognitum cog: **EWC Lifelong**

Learns new things without forgetting old lessons

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `ewc-lifelong` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"ewc-lifelong"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/ewc-lifelong/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/ewc-lifelong/logs?lines=5`) and report.

## Usage

```
/ewc-lifelong
/ewc-lifelong --once          # one-shot via /console with --once
/ewc-lifelong --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/ewc-lifelong --stop           # stop the cog on the seed
/ewc-lifelong --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"ewc-lifelong"}`
- `POST /api/v1/apps/ewc-lifelong/start`
- `POST /api/v1/apps/ewc-lifelong/stop`
- `POST /api/v1/apps/ewc-lifelong/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/ewc-lifelong/logs?lines=N`
- `GET  /api/v1/apps/ewc-lifelong/manifest`
- `GET  /api/v1/apps/ewc-lifelong/config`
- `PUT  /api/v1/apps/ewc-lifelong/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/ewc-lifelong/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/ewc-lifelong/`
