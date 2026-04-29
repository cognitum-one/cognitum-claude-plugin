---
description: Print a one-page status of the local Seed device
allowed-tools:
  - Bash
argument-hint: "[base-url]"
---

Read $COGNITUM_SEED_BASE (or use the first argument if provided, defaulting to http://169.254.42.1) and $COGNITUM_SEED_TOKEN.

Hit these endpoints in parallel and assemble a one-page status report:

| Endpoint | Field |
|---|---|
| `GET /api/v1/status` | firmware version, uptime, store size |
| `GET /api/v1/identity` | device UUID, Ed25519 pubkey fingerprint |
| `GET /api/v1/wifi/status` | SSID, IP, signal strength |
| `GET /api/v1/thermal/state` | CPU temp, governor, throttling |
| `GET /api/v1/store/status` | total vectors, dimension, file size |
| `GET /api/v1/firmware/status` | active slot (A/B), pending update |
| `GET /api/v1/pair/status` | pairing state, # active clients |

Use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```

Format as a clean ascii table or markdown table. Highlight in the output:
- 🔴 if CPU temp > 75°C or thermal throttling is active
- 🔴 if firmware has a pending update available
- 🟡 if WiFi signal < -70 dBm
- ✅ otherwise

Keep the response under 30 lines. No prose narration — just the table + warnings.
