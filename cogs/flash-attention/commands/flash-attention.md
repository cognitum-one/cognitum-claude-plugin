---
description: Install (if needed) and run the `flash-attention` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /flash-attention — Flash Attention

Cognitum cog: **Flash Attention**

Focuses sensing on specific areas for better accuracy

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `flash-attention` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"flash-attention"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/flash-attention/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/flash-attention/logs?lines=5`) and report.

## Usage

```
/flash-attention
/flash-attention --once          # one-shot via /console with --once
/flash-attention --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/flash-attention --stop           # stop the cog on the seed
/flash-attention --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"flash-attention"}`
- `POST /api/v1/apps/flash-attention/start`
- `POST /api/v1/apps/flash-attention/stop`
- `POST /api/v1/apps/flash-attention/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/flash-attention/logs?lines=N`
- `GET  /api/v1/apps/flash-attention/manifest`
- `GET  /api/v1/apps/flash-attention/config`
- `PUT  /api/v1/apps/flash-attention/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |
| `top_k` | integer | `4` | Number of highest-importance channels to focus on |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/flash-attention/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/flash-attention/`
