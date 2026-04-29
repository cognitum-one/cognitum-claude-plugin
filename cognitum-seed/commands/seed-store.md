---
description: Inspect or query the Seed vector store (RVF)
allowed-tools:
  - Bash
argument-hint: "stats | query <text> | get <vector-id> | neighbors <vector-id>"
---

Vector store operations on the local Seed.

**stats** (default) — `GET /api/v1/store/status`. Report: total vectors, embedding dimension, RVF file size on disk, current epoch, last write timestamp.

**query <text>** — POST `/api/v1/store/query` with `{"query": "<text>", "limit": 10}`. The Seed embeds the text locally and returns top-k matches with similarity scores + metadata. Format as a table. If the query returns 0 results, suggest the user check `seed-store stats` to confirm the store has any vectors.

**get <vector-id>** — `GET /api/v1/store/vectors/{id}`. Print the vector ID, full metadata JSON, embedding dimension, and a 6-number preview of the embedding (first 3, last 3, with `…` between).

**neighbors <vector-id>** — `GET /api/v1/store/graph/neighbors/{id}`. List up to 10 nearest neighbors with similarity scores.

Use:
```
curl -sk -H "Authorization: Bearer $COGNITUM_SEED_TOKEN" $COGNITUM_SEED_BASE<endpoint>
```

Truncate metadata strings > 80 chars in table output.
