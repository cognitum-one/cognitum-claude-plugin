---
description: Install (if needed) and run the `meta-adapt` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /meta-adapt — Meta Adapt

Cognitum cog: **Meta Adapt**

Automatically tunes itself for best performance

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `meta-adapt` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"meta-adapt"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/meta-adapt/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/meta-adapt/logs?lines=5`) and report.

## Usage

```
/meta-adapt
/meta-adapt --once          # one-shot via /console with --once
/meta-adapt --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/meta-adapt --stop           # stop the cog on the seed
/meta-adapt --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"meta-adapt"}`
- `POST /api/v1/apps/meta-adapt/start`
- `POST /api/v1/apps/meta-adapt/stop`
- `POST /api/v1/apps/meta-adapt/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/meta-adapt/logs?lines=N`
- `GET  /api/v1/apps/meta-adapt/manifest`
- `GET  /api/v1/apps/meta-adapt/config`
- `PUT  /api/v1/apps/meta-adapt/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between optimization cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/meta-adapt/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/meta-adapt/`
