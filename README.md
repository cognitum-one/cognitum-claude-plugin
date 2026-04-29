# cognitum-mcp

Claude Code plugin that registers the Cognitum MCP server.

## Install

```bash
/plugin marketplace add https://cognitum.one/marketplace.json
/plugin install cognitum-mcp
```

## What it does

Adds an MCP server named `cognitum` that connects to `https://cognitum.one/mcpSse` (SSE transport). The server exposes 7 tools:

| Tool | Purpose |
|------|---------|
| `health_check` | Liveness probe |
| `catalog_browse` | Browse Cognitum Seed product catalog |
| `lead_subscribe` | Subscribe an email to the notify-me / waitlist |
| `order_status` | Look up order status by email |
| `contact_send` | Send a contact-form message |
| `security_verify` | Run a security check on an input |
| `fleet_status` | Read fleet device status |
| `docs_search` | Search Cognitum docs |

## Manual install

If you'd rather skip the plugin, register directly:

```bash
claude mcp add cognitum https://cognitum.one/mcpSse
```

## Source

Source: https://github.com/cognitum-one/cognitum-claude-plugin
