---
description: Read the Seed thermal state and optionally toggle turbo
allowed-tools:
  - Bash
argument-hint: "state | telemetry | turbo on | turbo off"
---

Thermal subsystem queries.

Default to `state`.

**state** — `GET /api/v1/thermal/state`. Report CPU temp (°C), governor (powersave/ondemand/performance), throttle state, current frequency.

**telemetry** — `GET /api/v1/thermal/telemetry`. Print all sensor readings as a table.

**turbo on** | **turbo off** — POST `/api/v1/thermal/turbo` with `{"enabled":true}` or `{"enabled":false}`. Re-read state after, confirm change took effect. **Privileged** — requires the bearer token to have privileged scope; if you get a 403, tell the user the token they paired with does not have the `turbo` capability.

Use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```

Color-code the temperature in your output:
- 🟢 < 60°C
- 🟡 60–75°C
- 🔴 > 75°C
