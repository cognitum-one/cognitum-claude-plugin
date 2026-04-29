---
description: Pair Claude Code with a local Seed device and capture a bearer token
allowed-tools:
  - Bash
---

You are pairing this Claude Code instance with a local Cognitum Seed device.

The user has 60 seconds from the moment they physically open the pairing window on the Seed (button press or device admin UI). If they haven't done that yet, ask them to do it now and wait for confirmation.

When ready, do this:

1. Discover which transport works. Try in order:
   - `curl -sf http://169.254.42.1/api/v1/pair/status` (USB mode, port 80)
   - `curl -skf https://169.254.42.1:8443/api/v1/pair/status` (WiFi/LAN, port 8443, self-signed cert)
   - `curl -sf http://localhost:8444/api/v1/pair/status` (SSH-tunneled MCP transport)
   Report which one responded.

2. POST to `/api/v1/pair` on the working transport:
   ```
   curl -sk -X POST <BASE>/api/v1/pair \
     -H 'Content-Type: application/json' \
     -d '{"clientName": "Claude Code (CLI)"}'
   ```
   Capture the bearer token from the response.

3. Print the token to the user once and ask them to save it. Then suggest:
   ```
   export COGNITUM_SEED_TOKEN='<token>'
   export COGNITUM_SEED_BASE='<base-url>'
   ```
   These two env vars are read by the other `/seed-*` commands.

4. Verify with:
   ```
   curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE/api/v1/identity
   ```
   Confirm the device UUID and firmware version come back.

Stop after step 4. Do not proceed with WiFi setup or anything else unless the user asks.
