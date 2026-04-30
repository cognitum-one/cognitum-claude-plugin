# ADR: Beehive Monitor as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `beehive-monitor`
**Category**: building
**Canonical cog ADR**: [ADR-014 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-014-beehive-monitor.md)

## Context

This plugin wraps the `beehive-monitor` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, hum-band energy + chaos + piping autocorrelation classifier, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/beehive-monitor` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host with a microphone sited near the hive.

## Plugin invocation

- `/beehive-monitor` — install if needed, start, tail logs
- `/beehive-monitor --once` — one-shot via `/console` with `--once`
- `/beehive-monitor --console "..."` — pass arbitrary args
- `/beehive-monitor --stop` — stop the cog on the seed
- `/beehive-monitor --logs` — recent stdout/stderr

## RuView mode

None.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/beehive-monitor/`
