---
name: cog-operations
description: Day-2 operations for cogs in the fleet — log triage, health checks, OTA, rollback.
---

# Health check (single seed, over MCP)

```jsonc
seed.cogs.list()              // see what's installed and running
seed.cogs.logs({id, lines:50}) // recent stderr/stdout
seed.sensor.snapshot()        // is sensor input flowing?
seed.framework.identity()     // confirm seed is reachable + paired
seed.framework.firmware_status() // expected firmware version?
```

# Log triage

- Crash loops show as the cog appearing in `seed.cogs.list` but with no PID. Look at stderr in logs.
- Silent failures (no output) typically mean the cog can't reach its sensor source. Check `seed.sensor.snapshot` independently.
- Excess logs typically mean the cog forgot a debug `eprintln!`. Bump the version, ship a quieter build.

# Fleet rollout

The seed's cog binaries are pulled on-demand from `gs://cognitum-apps/cogs/arm/`. The OTA path for the agent itself is **separate** — `cognitum-ota.timer` (see `scripts/cognitum/ota-update.sh`). Cog updates don't go through the OTA path.

To force a fleet-wide cog update:
1. Bump the binary in GCS (see cog-deployment).
2. Per-seed: `seed.cogs.uninstall({id})` then `seed.cogs.install({id})` to force a fresh download. (Re-installing is the only path; install is idempotent for already-current SHAs.)

# Rollback

Re-upload the previous binary version under the same `cog-{id}-arm` path; uninstall + reinstall on affected seeds.

# Common incidents

| Symptom | Likely cause | Fix |
|---|---|---|
| `seed.cogs.list` 503 | seed agent restarting | Wait 30 s, retry |
| All cogs idle, sensor.snapshot empty | ESP32 disconnected | Check `seed.framework.mesh_status` and ESP32 wizard |
| New cog version not appearing | GCS cache | Upload again with `Cache-Control: no-cache` header explicitly |
| `seed.cogs.install` 404 | Wrong cog id or binary missing in GCS | `gsutil ls gs://cognitum-apps/cogs/arm/` |
