# ADR: Slip / Wet-Floor Zone as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `slip-fall-zone`
**Category**: industrial
**Canonical cog ADR**: [ADR-010 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-010-slip-fall-zone.md)

## Context

This plugin wraps the `slip-fall-zone` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, motion-variance + splash audio + cautious-gait fusion, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/slip-fall-zone` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host.

## Plugin invocation

- `/slip-fall-zone` — install if needed, start, tail logs
- `/slip-fall-zone --once` — one-shot via `/console` with `--once`
- `/slip-fall-zone --console "--ruview-mode"` — pass arbitrary args
- `/slip-fall-zone --stop` — stop the cog on the seed
- `/slip-fall-zone --logs` — recent stdout/stderr

## RuView mode

Optional. Pass `--ruview-mode` via `/slip-fall-zone --console "--ruview-mode"` to add cautious-gait CSI score to the risk fusion.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/slip-fall-zone/`
