---
description: Manage Seed WiFi — list current state, scan, or connect to a network
allowed-tools:
  - Bash
argument-hint: "status | scan | connect <SSID> <PSK>"
---

WiFi management for the Seed device.

Parse the user's argument (default to `status` if empty):

**status** — `GET /api/v1/wifi/status` and report SSID, IP, signal, link state.

**scan** — `GET /api/v1/wifi/scan` and print the list of nearby networks sorted by signal strength. Columns: SSID, signal (dBm), security (WPA2/WPA3/Open).

**connect <SSID> <PSK>** — POST `/api/v1/wifi/connect` with `{"ssid":"<SSID>","psk":"<PSK>"}`. After the call, poll `/api/v1/wifi/status` every 2 seconds for up to 15 seconds until either:
- IP is assigned (success — print the new IP and link state)
- 15 seconds elapse (timeout — print last-known state and suggest re-checking signal strength)

In all cases use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```

Treat the PSK as sensitive — do not echo it in any subsequent output or logs.
