# ADR: Glass-Break Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `glass-break`
**Category**: security
**Canonical cog ADR**: [ADR-006 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-006-glass-break.md)

## Context

This plugin wraps the `glass-break` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, two-phase bang+shatter detector, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/glass-break` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host.

## Plugin invocation

- `/glass-break` — install if needed, start, tail logs
- `/glass-break --once` — one-shot via `/console` with `--once`
- `/glass-break --console "--ruview-mode"` — pass arbitrary args
- `/glass-break --stop` — stop the cog on the seed
- `/glass-break --logs` — recent stdout/stderr

## RuView mode

Optional. Pass `--ruview-mode` via `/glass-break --console "--ruview-mode"` to enable CSI-based reinforcement.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/glass-break/`
