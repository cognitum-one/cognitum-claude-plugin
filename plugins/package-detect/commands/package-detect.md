---
description: Install (if needed) and run the `package-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /package-detect — Package Arrival Detection

Cognitum cog: **Package Arrival Detection**

Sustained CSI-shift detector for porch / loading bay package arrivals and departures. Requires ESP32 CSI ruview input

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `package-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"package-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/package-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/package-detect/logs?lines=5`) and report.

## Usage

```
/package-detect
/package-detect --once          # one-shot via /console with --once
/package-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/package-detect --stop           # stop the cog on the seed
/package-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"package-detect"}`
- `POST /api/v1/apps/package-detect/start`
- `POST /api/v1/apps/package-detect/stop`
- `POST /api/v1/apps/package-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/package-detect/logs?lines=N`
- `GET  /api/v1/apps/package-detect/manifest`
- `GET  /api/v1/apps/package-detect/config`
- `PUT  /api/v1/apps/package-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `2` | Sampling interval |
| `persistence` | integer | `30` | Seconds the shift must persist to count as package arrival |
| `shift_z` | float | `2.5` | Z-score of CSI variance shift to enter transient state |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/package-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/package-detect/`
