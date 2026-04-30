---
description: Install (if needed) and run the `forklift-proximity` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /forklift-proximity — Forklift Proximity

Cognitum cog: **Forklift Proximity**

Warns if a forklift gets too close to workers

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `forklift-proximity` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"forklift-proximity"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/forklift-proximity/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/forklift-proximity/logs?lines=5`) and report.

## Usage

```
/forklift-proximity
/forklift-proximity --once          # one-shot via /console with --once
/forklift-proximity --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/forklift-proximity --stop           # stop the cog on the seed
/forklift-proximity --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"forklift-proximity"}`
- `POST /api/v1/apps/forklift-proximity/start`
- `POST /api/v1/apps/forklift-proximity/stop`
- `POST /api/v1/apps/forklift-proximity/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/forklift-proximity/logs?lines=N`
- `GET  /api/v1/apps/forklift-proximity/manifest`
- `GET  /api/v1/apps/forklift-proximity/config`
- `PUT  /api/v1/apps/forklift-proximity/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/forklift-proximity/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/forklift-proximity/`
