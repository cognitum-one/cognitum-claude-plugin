---
description: Check for + apply Seed firmware updates (OTA)
allowed-tools:
  - Bash
argument-hint: "check | apply"
---

OTA firmware management for the Seed device. Default action: `check`.

**check** — `GET /api/v1/upgrade/check`. Report:
- current version (from `/api/v1/firmware/status`)
- available version (from check response)
- size, signature status, release notes URL if present
- A/B slot state (which slot is active, which is the staging target)

**apply** — Confirm with the user FIRST (this is a dangerous operation that reboots the device).
After explicit confirmation:
1. POST `/api/v1/firmware/update` to start the download.
2. Poll `/api/v1/upgrade/status` every 5 seconds. Print progress %.
3. When status is `staged`, POST `/api/v1/upgrade/apply`.
4. Device will reboot. Tell the user to wait ~60 seconds, then run `/seed-status` to verify the new version is active.

Use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```

Never run `apply` without an explicit "yes apply" or similar confirmation in the same turn. If unsure, ask.
