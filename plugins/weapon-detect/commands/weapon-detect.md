---
description: Install (if needed) and run the `weapon-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /weapon-detect — Weapon Detection

Cognitum cog: **Weapon Detection**

Detects concealed metal objects on a person

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `weapon-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"weapon-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/weapon-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/weapon-detect/logs?lines=5`) and report.

## Usage

```
/weapon-detect
/weapon-detect --once          # one-shot via /console with --once
/weapon-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/weapon-detect --stop           # stop the cog on the seed
/weapon-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"weapon-detect"}`
- `POST /api/v1/apps/weapon-detect/start`
- `POST /api/v1/apps/weapon-detect/stop`
- `POST /api/v1/apps/weapon-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/weapon-detect/logs?lines=N`
- `GET  /api/v1/apps/weapon-detect/manifest`
- `GET  /api/v1/apps/weapon-detect/config`
- `PUT  /api/v1/apps/weapon-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Seconds between detection scans |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/weapon-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/weapon-detect/`
