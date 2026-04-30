# ADR: Gunshot Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `gunshot-detect`
**Category**: security
**Canonical cog ADR**: [ADR-007 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-007-gunshot-detect.md)

## Context

This plugin wraps the `gunshot-detect` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, saturating peak + exponential decay detector, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/gunshot-detect` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host.

## Plugin invocation

- `/gunshot-detect` — install if needed, start, tail logs
- `/gunshot-detect --once` — one-shot via `/console` with `--once`
- `/gunshot-detect --console "--ruview-mode"` — pass arbitrary args
- `/gunshot-detect --stop` — stop the cog on the seed
- `/gunshot-detect --logs` — recent stdout/stderr

## RuView mode

Optional. Pass `--ruview-mode` via `/gunshot-detect --console "--ruview-mode"` to enable CSI motion-drop reinforcement in a 5s post-peak window.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/gunshot-detect/`
