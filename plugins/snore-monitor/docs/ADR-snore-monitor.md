# ADR: Snore Monitor as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `snore-monitor`
**Category**: health
**Canonical cog ADR**: [ADR-005 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-005-snore-monitor.md)

## Context

This plugin wraps the `snore-monitor` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, periodic low-band energy tracking approach, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/snore-monitor` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host.

## Plugin invocation

- `/snore-monitor` — install if needed, start, tail logs
- `/snore-monitor --once` — one-shot via `/console` with `--once`
- `/snore-monitor --console "--ruview-mode"` — pass arbitrary args
- `/snore-monitor --stop` — stop the cog on the seed
- `/snore-monitor --logs` — recent stdout/stderr

## RuView mode

Optional. Pass `--ruview-mode` via `/snore-monitor --console "--ruview-mode"` to enable CSI-based reinforcement.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/snore-monitor/`
