---
description: Install (if needed) and run the `ppe-compliance` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /ppe-compliance — PPE Compliance

Cognitum cog: **PPE Compliance**

Cog-composition layer: alerts when ruview-densepose detects presence in a restricted zone without an accompanying PPE-camera-cog confirmation vector

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `ppe-compliance` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"ppe-compliance"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/ppe-compliance/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/ppe-compliance/logs?lines=5`) and report.

## Usage

```
/ppe-compliance
/ppe-compliance --once          # one-shot via /console with --once
/ppe-compliance --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/ppe-compliance --stop           # stop the cog on the seed
/ppe-compliance --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"ppe-compliance"}`
- `POST /api/v1/apps/ppe-compliance/start`
- `POST /api/v1/apps/ppe-compliance/stop`
- `POST /api/v1/apps/ppe-compliance/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/ppe-compliance/logs?lines=N`
- `GET  /api/v1/apps/ppe-compliance/manifest`
- `GET  /api/v1/apps/ppe-compliance/config`
- `PUT  /api/v1/apps/ppe-compliance/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `5` | Sampling interval |
| `zone` | string | `restricted` | Restricted zone label |
| `confirmation_window` | integer | `60` | Seconds within which PPE confirmation must be observed |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/ppe-compliance/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/ppe-compliance/`
