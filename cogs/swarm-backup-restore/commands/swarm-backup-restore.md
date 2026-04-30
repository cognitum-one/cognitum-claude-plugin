---
description: Install (if needed) and run the `swarm-backup-restore` cog on the paired seed
allowed-tools: Bash, WebFetch, Read
---

# /swarm-backup-restore — Swarm Backup & Restore

Cognitum cog: **Swarm Backup & Restore**

Auto-backup data to other seeds — one-click restore

## What this command does

When invoked, it will:

1. Verify the seed is reachable (USB at `http://169.254.42.1` or LAN HTTPS at `https://169.254.42.1:8443`).
2. Check whether `swarm-backup-restore` is already installed (`GET /api/v1/apps`).
3. If not installed, install it (`POST /api/v1/apps/install` with `{"id":"swarm-backup-restore"}`) — the seed downloads the ARM binary from the cog registry on GCS.
4. Start it if not already running (`POST /api/v1/apps/swarm-backup-restore/start`).
5. Read the most recent JSON output (`GET /api/v1/apps/swarm-backup-restore/logs?lines=5`) and report.

## Usage

```
/swarm-backup-restore
/swarm-backup-restore --once          # one-shot via /console with --once
/swarm-backup-restore --console "..."  # pass arbitrary args (--ruview-mode, --interval N, etc.)
/swarm-backup-restore --stop           # stop the cog on the seed
/swarm-backup-restore --logs           # tail the cog's recent output
```

## Seed endpoint reference

- `POST /api/v1/apps/install`               body: `{"id":"swarm-backup-restore"}`
- `POST /api/v1/apps/swarm-backup-restore/start`
- `POST /api/v1/apps/swarm-backup-restore/stop`
- `POST /api/v1/apps/swarm-backup-restore/console`         body: `{"command":"--once"}`
- `GET  /api/v1/apps/swarm-backup-restore/logs?lines=N`
- `GET  /api/v1/apps/swarm-backup-restore/manifest`
- `GET  /api/v1/apps/swarm-backup-restore/config`
- `PUT  /api/v1/apps/swarm-backup-restore/config`          body: keyed config knobs

## Resource budget

- Pi Zero 2 W: target binary < 500 KB armhf.
- Seed enforces max 3 concurrent active cogs — stop another cog if needed.

## Configuration knobs

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `interval` | integer | `120` | Seconds between backup cycles |
| `peer` | string | `169.254.42.1` | IP address of the peer seed to backup to or restore from |
| `mode` | select | `backup` | Whether to backup local data to peer or restore from peer |

## See also

- Cog source: `cognitum-one/cogs` repo, `src/cogs/swarm-backup-restore/`
- ADR: see `docs/adrs/` in the cogs repo for design notes.
- Marketplace plugin source: `cognitum-one/cognitum-claude-plugin/cogs/swarm-backup-restore/`
