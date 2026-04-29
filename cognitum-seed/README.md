# cognitum-seed

Claude Code plugin for managing a local **Cognitum Seed** appliance.

## What it does

Connects Claude Code to a Seed device on your network (link-local at `169.254.42.1` over USB-C, or any LAN IP via WiFi) and gives you:

- **`cognitum-seed` MCP server** — exposes the device's 114 MCP tools (vector store, sensors, thermal, OTA, witness chain, cognitive container, …)
- **8 slash commands** for common operations
- **`seed-installer` agent** for guided first-time setup
- **`seed-troubleshoot` skill** Claude reaches for when you describe a Seed problem

## Install

```
/plugin marketplace add https://cognitum.one/marketplace.json
/plugin install cognitum-seed
```

After install, restart Claude Code. Then either:
- **First-time setup**: `/seed-pair` (or invoke the `seed-installer` agent for the full guided flow)
- **Already paired**: set `COGNITUM_SEED_TOKEN` and `COGNITUM_SEED_BASE` env vars and try `/seed-status`

## Slash commands

| Command | Purpose |
|---|---|
| `/seed-pair` | Pair Claude Code with a Seed and capture a bearer token |
| `/seed-status` | One-page device summary (firmware, WiFi, thermal, store, sensors) |
| `/seed-wifi [status\|scan\|connect SSID PSK]` | Manage WiFi |
| `/seed-ota [check\|apply]` | Check or apply firmware updates (OTA) |
| `/seed-thermal [state\|telemetry\|turbo on\|turbo off]` | Thermal subsystem |
| `/seed-store [stats\|query <text>\|get <id>\|neighbors <id>]` | Vector store / RVF |
| `/seed-snapshot [output-file]` | Capture cognitive snapshot for debugging |
| `/seed-cluster` | Multi-Seed cluster + peer health |

## MCP server

The plugin's `.mcp.json` registers an MCP server named `cognitum-seed`:

| Mode | URL | Auth |
|---|---|---|
| **USB-C gadget** (default) | `http://169.254.42.1/mcp` | none (trusted via cable) |
| **WiFi / LAN** | `https://<seed-ip>:8443/mcp` | `Authorization: Bearer ${COGNITUM_SEED_TOKEN}` |

To switch modes, set `COGNITUM_SEED_MCP_URL` in your env before starting Claude Code.

After install + restart, run `/mcp` and you should see `cognitum-seed` connected with ~24 tools (default scope) or up to 114 (full scope, configured device-side).

## Auth model (TL;DR)

The Seed uses per-client bearer tokens (RFC 4648 base64url). To get one:

1. Physically open the pairing window (button hold for 3s, LED → blue) — 60-second window.
2. POST `/api/v1/pair` with a client name; receive a token.
3. All subsequent requests: `Authorization: Bearer <token>`.

Tokens are persistent. Revoke with `DELETE /api/v1/pair/{clientId}`.

USB gadget mode (port 80) requires no auth — physical cable counts as the trust anchor.

## Endpoints used by this plugin

| Group | Endpoints |
|---|---|
| Pairing | `POST /pair`, `GET /pair/status`, `GET /pair/clients` |
| System | `GET /status`, `GET /identity`, `GET /security/status` |
| WiFi | `GET /wifi/status`, `GET /wifi/scan`, `POST /wifi/connect` |
| Firmware | `GET /firmware/status`, `GET /upgrade/check`, `POST /firmware/update`, `POST /upgrade/apply` |
| Thermal | `GET /thermal/state`, `GET /thermal/telemetry`, `POST /thermal/turbo` |
| Store | `GET /store/status`, `POST /store/query`, `GET /store/vectors/{id}`, `GET /store/graph/neighbors/{id}` |
| Cognitive | `GET /cognitive/snapshot`, `GET /cognitive/status`, `GET /coherence/profile` |
| Witness | `GET /witness/chain`, `POST /witness/verify` |
| Cluster | `GET /cluster/health`, `GET /peers`, `GET /sync/stats` |

All under base `https://169.254.42.1:8443/api/v1/` (or `http://169.254.42.1/api/v1/` over USB).

## Source

`https://github.com/ruvnet/cognitum-claude-plugin/tree/main/cognitum-seed`

The cloud-side `cognitum-mcp` plugin (registers `https://cognitum.one/mcpSse`) is in the same repo's root.
