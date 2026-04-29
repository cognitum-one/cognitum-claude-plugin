---
name: seed-troubleshoot
description: Diagnose common Cognitum Seed device issues — won't pair, no WiFi, firmware stuck, MCP tools not appearing in Claude Code. Use when the user reports a Seed-related symptom and asks "why isn't this working?".
---

# Seed Troubleshooting Playbook

When a user has trouble with a Seed device, work through these symptom→cause checks in order. Don't run them blindly — ask for the symptom first, then jump to the matching section.

## Symptom: "Pairing window timed out / 401 on POST /pair"

Causes (most likely first):
1. **Window not opened** — user must hold the device button for 3s until LED turns blue. Pairing only accepts unauth POSTs during this window.
2. **Window expired** — 60s from button press. Re-open and retry.
3. **Wrong transport** — if device is on WiFi, must use `https://<ip>:8443`. If on USB-C only, use `http://169.254.42.1`. Confirm with `/seed-pair` discovery step.
4. **Already paired and out of slots** — `GET /api/v1/pair/clients`. The Seed has a max client cap (typically 16). DELETE old entries with `/api/v1/pair/{clientId}`.

## Symptom: "Bearer token returns 403 on every endpoint"

Causes:
1. **Token revoked** — check `GET /api/v1/pair/clients` for the token's clientId; if absent, re-pair.
2. **mTLS required for that endpoint** — some endpoints (firmware, security) require client cert in addition to bearer. Check the endpoint docs in `/guide` on the device.
3. **Privileged scope missing** — bearer tokens have read/write/dangerous tiers. Re-pair from a privileged context (USB cable) to get a higher-tier token.

## Symptom: "WiFi connect returns 200 but never gets an IP"

Causes:
1. **Wrong PSK** — silent failure. Re-check by re-running `/seed-wifi connect <SSID> <PSK>` and confirming.
2. **Signal too weak** — `GET /api/v1/wifi/scan` and look at the SSID's signal. Below -75dBm is unreliable.
3. **AP isolation / captive portal** — corporate WiFi or guest networks often block headless devices. Move to a normal home WiFi to test.
4. **Hostname conflict** — if a previous Seed used the same MAC/hostname, the AP may have it blacklisted. Reboot AP or change MAC.

## Symptom: "OTA download starts but stuck at <100%"

Causes:
1. **Slow upstream** — firmware bundles are 100–400 MB. Watch `/api/v1/upgrade/status` for `bytes_received` to confirm it's making progress.
2. **GCS bucket throttled** — Seed pulls from `gs://cognitum-firmware/`. If multiple devices update simultaneously, throughput drops.
3. **Disk full** — `GET /api/v1/firmware/status` for `staging_slot.free_bytes`. Need ~2x bundle size free.
4. **Signature verification failed** — log line "build-key mismatch" means the device's measured-boot key doesn't trust the new bundle. Don't override — escalate.

## Symptom: "MCP tools from cognitum-seed don't appear in /mcp"

Causes:
1. **Plugin disabled** — `/plugin list`, confirm `cognitum-seed` shows ● (filled circle). If ◯, enable + restart Claude Code.
2. **Wrong URL in .mcp.json** — default is `http://169.254.42.1/mcp` (USB). For WiFi, override env: `export COGNITUM_SEED_MCP_URL=https://<wifi-ip>:8443/mcp`.
3. **Self-signed cert rejected** — Claude Code may refuse the Seed's self-signed cert on HTTPS. Workarounds: use USB-mode HTTP, or add the Seed's cert to your system trust store.
4. **Pairing token not in env** — for HTTPS WiFi mode, set `COGNITUM_SEED_TOKEN`. The .mcp.json passes it as `Authorization: Bearer ...`.
5. **Device offline** — `ping 169.254.42.1` (USB) or the LAN IP. If unreachable, the issue is below the API layer.

## Symptom: "/seed-status shows 🔴 thermal warning"

Causes:
1. **Sustained heavy load** — check `/api/v1/cognitive/status` for tick rate. Throttle by lowering tick rate or disabling turbo (`/seed-thermal turbo off`).
2. **Enclosure airflow blocked** — physical fix; no API help.
3. **Silicon variation** — `/api/v1/thermal/silicon-profile` shows the BCM SoC's characterized envelope. If accuracy is < 0.85, recharacterize: `POST /api/v1/thermal/characterize`.

## Symptom: "Witness chain has gaps / verification fails"

This is serious — possible custody breach. Stop normal operations and:
1. `GET /api/v1/witness/chain?limit=100` — find the gap.
2. `POST /api/v1/witness/verify` on the entries adjacent to the gap.
3. If gaps confirm, escalate to ops. Do NOT auto-repair the chain — that erases the breach evidence.

# General debugging endpoints (always safe to call)

- `GET /api/v1/status` — alive/firmware
- `GET /api/v1/security/status` — auth posture
- `GET /api/v1/boot/measurements` — measured boot chain
- `GET /api/v1/delta/history?limit=50` — recent state changes (audit log)

Use:
```bash
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```
