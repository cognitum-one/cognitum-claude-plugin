---
description: Install (if needed) and run the `self-healing-mesh` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /self-healing-mesh — Self-Healing Mesh

Cognitum cog: **Self-Healing Mesh**

Keeps sensor mesh running even when nodes drop out

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `self-healing-mesh` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"self-healing-mesh"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/self-healing-mesh/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/self-healing-mesh/logs?lines=5`) and report.

## Usage

```
/self-healing-mesh
/self-healing-mesh --once          # one-shot via /console with --once
/self-healing-mesh --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/self-healing-mesh --stop           # stop the cog on the seed
/self-healing-mesh --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"self-healing-mesh"}`
- `POST /api/v1/apps/self-healing-mesh/start`
- `POST /api/v1/apps/self-healing-mesh/stop`
- `POST /api/v1/apps/self-healing-mesh/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/self-healing-mesh/logs?lines=N`
- `GET  /api/v1/apps/self-healing-mesh/manifest`
- `GET  /api/v1/apps/self-healing-mesh/config`
- `PUT  /api/v1/apps/self-healing-mesh/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between mesh health checks |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/self-healing-mesh/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/self-healing-mesh/`
