---
description: Take a cognitive snapshot — current Seed mind state for debugging or analysis
allowed-tools:
  - Bash
  - Write
argument-hint: "[output-file]"
---

Capture the Seed's current cognitive container state.

1. `GET /api/v1/cognitive/snapshot` — full snapshot JSON.
2. `GET /api/v1/cognitive/status` — container resources (memory, ticks, last update).
3. `GET /api/v1/coherence/profile` — temporal coherence metrics.
4. `GET /api/v1/witness/chain?limit=20` — last 20 witness entries.
5. `GET /api/v1/sensor/embedding/latest` — latest sensor feature embedding.

Bundle into one JSON object:
```json
{
  "captured_at": "<ISO timestamp>",
  "device_uuid": "...",
  "snapshot": {...},
  "container": {...},
  "coherence": {...},
  "witness_recent": [...],
  "sensor_embedding": {...}
}
```

If the user provided an output file path as $1, write the bundle there with `Write`. Otherwise print a summary (5-10 lines) of the most interesting fields and offer to dump the full bundle.

Use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```

If `/api/v1/cognitive/*` returns 404, the device build does not have the `cognitive-container` feature compiled in — say so explicitly and stop.
