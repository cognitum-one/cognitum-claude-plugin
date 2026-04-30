---
description: Install (if needed) and run the `rag-local` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /rag-local — Local RAG Engine

Cognitum cog: **Local RAG Engine**

Search your documents using AI — runs on the seed

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `rag-local` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"rag-local"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/rag-local/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/rag-local/logs?lines=5`) and report.

## Usage

```
/rag-local
/rag-local --once          # one-shot via /console with --once
/rag-local --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/rag-local --stop           # stop the cog on the seed
/rag-local --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"rag-local"}`
- `POST /api/v1/apps/rag-local/start`
- `POST /api/v1/apps/rag-local/stop`
- `POST /api/v1/apps/rag-local/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/rag-local/logs?lines=N`
- `GET  /api/v1/apps/rag-local/manifest`
- `GET  /api/v1/apps/rag-local/config`
- `PUT  /api/v1/apps/rag-local/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between index refresh cycles |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/rag-local/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/rag-local/`
