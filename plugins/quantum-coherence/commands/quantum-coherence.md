---
description: Install (if needed) and run the `quantum-coherence` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /quantum-coherence — Quantum Coherence

Cognitum cog: **Quantum Coherence**

Quantum-inspired model for advanced signal states

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `quantum-coherence` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"quantum-coherence"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/quantum-coherence/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/quantum-coherence/logs?lines=5`) and report.

## Usage

```
/quantum-coherence
/quantum-coherence --once          # one-shot via /console with --once
/quantum-coherence --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/quantum-coherence --stop           # stop the cog on the seed
/quantum-coherence --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"quantum-coherence"}`
- `POST /api/v1/apps/quantum-coherence/start`
- `POST /api/v1/apps/quantum-coherence/stop`
- `POST /api/v1/apps/quantum-coherence/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/quantum-coherence/logs?lines=N`
- `GET  /api/v1/apps/quantum-coherence/manifest`
- `GET  /api/v1/apps/quantum-coherence/config`
- `PUT  /api/v1/apps/quantum-coherence/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `10` | Seconds between measurements |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/quantum-coherence/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/quantum-coherence/`
