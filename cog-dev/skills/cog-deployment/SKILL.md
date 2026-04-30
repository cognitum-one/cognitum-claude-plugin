---
name: cog-deployment
description: Ship a cog to the fleet — GCS upload, registry bump, install verification.
---

# Pipeline

1. **Build (release, ARM)** — same as cog-testing, but mandatory.
2. **Compute SHA-256** — `sha256sum cog-{id}` — record for the registry.
3. **Upload binary** — `gsutil -h "Cache-Control:no-cache,no-store,must-revalidate" cp cog-{id} gs://cognitum-apps/cogs/arm/cog-{id}-arm`
4. **Bump app-registry.json** — pull existing JSON, update version + size + sha256 for `{id}`, push back to `gs://cognitum-apps/app-registry.json` (also with no-cache headers).
5. **Verify on a seed** — `seed.cogs.install({id})` — should fetch the new binary and report the new SHA.

# Order matters

Always upload the binary **before** bumping the registry. If the registry says v1.2.0 exists but the binary is still v1.1.0 in GCS, every seed that polls the registry will 404.

# Cache invalidation

The infamous one. GCS objects without explicit `Cache-Control: no-cache` get cached at edge POPs for hours. Always set the header on **every** upload.

# Rollback

If a seed reports a problem after install, the seed's local cache holds the previous binary. Rollback = re-upload the previous version under the same path. Seeds will pull on next install.

# What "passing" means

- `gsutil ls -L gs://cognitum-apps/cogs/arm/cog-{id}-arm` shows new size + new metadata.crc32c
- `app-registry.json` `{id}` entry's `version`, `size`, and `sha256` all reflect the new build.
- A test seed's `seed.cogs.install({id})` returns `{"status":"installed"}` and `seed.cogs.list` shows the new SHA in the cog's manifest.
