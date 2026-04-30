---
name: cog-deploy
description: Cross-compile, upload to GCS, bump the registry, and confirm rollout to a target seed via MCP.
arguments:
  - name: id
    description: Cog id to deploy
    required: true
---

# Steps

1. Load the `cog-deployment` skill.
2. Spawn the `cog-deployer` subagent with `{id}`.
3. The deployer runs (in order):
   - `cargo build -p cog-{id} --release --target arm-unknown-linux-gnueabihf` from `external/cogs/`
   - Compute SHA-256 of the artifact
   - Upload `target/arm-unknown-linux-gnueabihf/release/cog-{id}` to `gs://cognitum-apps/cogs/arm/cog-{id}-arm` (with `Cache-Control: no-cache`)
   - Update `gs://cognitum-apps/app-registry.json` to bump the version + size for `{id}`
   - Over MCP: `seed.cogs.install({id})` to verify the new binary lands
4. Report the artifact SHA, GCS URL, and seed install confirmation.

# Constraints

- Never bump the registry without uploading the binary first (otherwise seeds will 404 on install).
- Always set Cache-Control on the GCS object — prior incidents had stale caches serving outdated binaries for hours.
- If the user is not authenticated to gcloud, refuse and explain.
- This deploys to the **fleet** registry. For seed-only testing, prefer `/cog-test`.
