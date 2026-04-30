# ADR: Smoke / Fire Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `smoke-fire`
**Category**: building
**Canonical cog ADR**: [ADR-012 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-012-smoke-fire.md)

## Context

This plugin wraps the `smoke-fire` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, multi-signal acoustic + thermal-drift fusion, regulatory caveat, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/smoke-fire` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host. Not a UL-listed replacement for code-required smoke alarms.

## Plugin invocation

- `/smoke-fire` — install if needed, start, tail logs
- `/smoke-fire --once` — one-shot via `/console` with `--once`
- `/smoke-fire --console "--ruview-mode"` — pass arbitrary args
- `/smoke-fire --stop` — stop the cog on the seed
- `/smoke-fire --logs` — recent stdout/stderr

## RuView mode

Optional. Pass `--ruview-mode` via `/smoke-fire --console "--ruview-mode"` to add CSI plume signature as a third evidence stream.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/smoke-fire/`
