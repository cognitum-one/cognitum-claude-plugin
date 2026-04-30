# ADR: Frost Warning as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `frost-warning`
**Category**: building
**Canonical cog ADR**: [ADR-013 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-013-frost-warning.md)

## Context

This plugin wraps the `frost-warning` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, temperature trend + dewpoint depression projection, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/frost-warning` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host with a temperature/humidity sensor attached.

## Plugin invocation

- `/frost-warning` — install if needed, start, tail logs
- `/frost-warning --once` — one-shot via `/console` with `--once`
- `/frost-warning --console "..."` — pass arbitrary args
- `/frost-warning --stop` — stop the cog on the seed
- `/frost-warning --logs` — recent stdout/stderr

## RuView mode

None.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/frost-warning/`
