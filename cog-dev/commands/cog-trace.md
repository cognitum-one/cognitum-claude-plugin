---
name: cog-trace
description: Tail a cog's stdout/stderr from the seed over MCP. Useful for debugging without SSH.
arguments:
  - name: id
    description: Cog id to trace
    required: true
---

# Steps

1. Over MCP `cognitum-seed`, call `seed.cogs.logs({id, lines: 100})` and display the result.
2. Loop: every 2 s, re-call with `lines: 20` and append new lines.
3. Stop when the user interrupts (Ctrl-C / cancel).

# Notes

- This isn't a true streaming tail (no SSE on `seed.cogs.logs` yet); it's a poll. Acceptable since cog logs don't typically exceed a few lines per second.
- If `seed.cogs.logs` returns "cog not running", confirm with `seed.cogs.list` and offer to start it.
