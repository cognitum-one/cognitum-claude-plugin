---
description: Install (if needed) and run the `prompt-shield` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /prompt-shield — Prompt Shield

Cognitum cog: **Prompt Shield**

Blocks signal replay and injection attacks on the seed

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `prompt-shield` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"prompt-shield"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/prompt-shield/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/prompt-shield/logs?lines=5`) and report.

## Usage

```
/prompt-shield
/prompt-shield --once          # one-shot via /console with --once
/prompt-shield --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/prompt-shield --stop           # stop the cog on the seed
/prompt-shield --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"prompt-shield"}`
- `POST /api/v1/apps/prompt-shield/start`
- `POST /api/v1/apps/prompt-shield/stop`
- `POST /api/v1/apps/prompt-shield/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/prompt-shield/logs?lines=N`
- `GET  /api/v1/apps/prompt-shield/manifest`
- `GET  /api/v1/apps/prompt-shield/config`
- `PUT  /api/v1/apps/prompt-shield/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `2` | Seconds between security scans |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/prompt-shield/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/prompt-shield/`
