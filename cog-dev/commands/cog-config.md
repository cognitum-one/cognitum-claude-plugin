---
name: cog-config
description: Read or update a cog's configuration over MCP, schema-driven from cog.toml [config].
arguments:
  - name: id
    description: Cog id
    required: true
  - name: change
    description: Optional inline JSON object to apply (e.g. `{"interval":15}`); when omitted, displays current config and prompts.
    required: false
---

# Steps

1. Over MCP `cognitum-seed`, call `seed.cogs.config_get({id})` and display the current values + the schema (from `cognitum://cogs/{id}/manifest`).
2. If `{change}` was provided, call `seed.cogs.config_set({id, config: <merged>})` and confirm.
3. If `{change}` is omitted, prompt the user for keys to update and apply iteratively.

# Notes

- The seed validates min/max/type per the cog's `cog.toml [config]` block — invalid values are rejected with a clear message.
- Config changes apply on next start; offer the user `seed.cogs.stop` + `seed.cogs.start` to apply now.
