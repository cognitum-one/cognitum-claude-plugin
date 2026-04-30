---
name: seed-mcp-quickstart
description: How to wire a Claude Code session to a Cognitum seed's MCP endpoint and tour the tool palette.
---

# Connect to the seed

```bash
# USB gadget (default)
cog-dev mcp connect

# WiFi seed
cog-dev mcp connect --seed http://192.168.1.106 --bearer "$SEED_TOKEN"

# Auto-discover
cog-dev mcp connect --auto-discover
```

Internally this wraps `claude mcp add cognitum-seed` with the bundled streamable-HTTP transport, so users only see the one-liner above. Auto-discovery probes USB (`169.254.42.1`), mDNS (`cognitum.local`), and configured WiFi seed addresses in order.

# Tool palette (per ADR-092)

| Group | Tools |
|---|---|
| `seed.framework.*` | `identity`, `firmware_status`, `mesh_status`, `esp32_flash_url` (Public — no auth needed) |
| `seed.cogs.*` | `list`, `available`, `install`, `start`, `stop`, `logs`, `config_get`, `config_set`, `console`, `uninstall` |
| `seed.sensor.*` | `snapshot`, `config`, `list` |
| `seed.rvf.*` | `query`, `container.list`, `kernel.list`, `kernel.call` |
| `seed.witness.*` | `chain`, `verify` |
| `seed.swarm.*` | `status`, `peers.list` |

# Resources

| URI | Use |
|---|---|
| `cognitum://registry` | Live 88-cog catalog |
| `cognitum://identity` | Device UUID + key + paired flag |
| `cognitum://sensor/stream/snapshot` | Latest sensor frame |
| `cognitum://rvf/witness/head` | Witness chain head |
| `cognitum://firmware/esp32/latest` | ESP32 firmware metadata |
| `cognitum://cogs/{id}/manifest` | Per-cog manifest (with [config] schema) |
| `cognitum://cogs/{id}/logs/recent` | Per-cog recent logs |

# Common patterns

- **Building a UI for a cog**: install with `seed.cogs.install`, fetch its manifest from `cognitum://cogs/{id}/manifest` to render config controls, stream output via `seed.cogs.logs`.
- **Authoring a new cog**: `/cog-new <id>` scaffolds, `/cog-test <id>` validates, `/cog-deploy <id>` ships.
- **Debugging**: `/cog-trace <id>` tails logs without SSH.
- **Fleet operations**: cross-cuts via the seed's swarm group; not all seeds need to be reachable directly — use mesh.

# Auth model

- **Public** tools (`seed.framework.*`, `seed.guide.*`, `seed.cogs.list/available`) need no token.
- **Paired** tools (everything that mutates) require pairing. Pair via the cog-store UI or `POST /api/v1/pair` with the cog-store ceremony.
- **Privileged** tools (firmware update, etc.) require pairing **plus** an explicit policy grant.
