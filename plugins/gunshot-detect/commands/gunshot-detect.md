---
description: Install (if needed) and run the `gunshot-detect` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /gunshot-detect — Gunshot Detection

Cognitum cog: **Gunshot Detection**

Saturating peak + exponential decay acoustic detector with optional ruview CSI motion-drop reinforcement

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `gunshot-detect` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"gunshot-detect"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/gunshot-detect/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/gunshot-detect/logs?lines=5`) and report.

## Usage

```
/gunshot-detect
/gunshot-detect --once          # one-shot via /console with --once
/gunshot-detect --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/gunshot-detect --stop           # stop the cog on the seed
/gunshot-detect --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"gunshot-detect"}`
- `POST /api/v1/apps/gunshot-detect/start`
- `POST /api/v1/apps/gunshot-detect/stop`
- `POST /api/v1/apps/gunshot-detect/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/gunshot-detect/logs?lines=N`
- `GET  /api/v1/apps/gunshot-detect/manifest`
- `GET  /api/v1/apps/gunshot-detect/config`
- `PUT  /api/v1/apps/gunshot-detect/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `1` | Sampling interval |
| `peak_threshold` | float | `0.95` | Normalized amplitude above which saturation gate fires |
| `decay_frames` | integer | `4` | Frames in which exponential decay must match |
| `cooldown` | integer | `30` | Cooldown after fire |
| `ruview_mode` | boolean | `False` | Reinforce detection with CSI motion-drop evidence in 5s post-peak window |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/gunshot-detect/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/gunshot-detect/`
