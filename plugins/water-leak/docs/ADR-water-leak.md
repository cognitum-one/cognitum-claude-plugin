# ADR: Water Leak Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `water-leak`
**Category**: building
**Canonical cog ADR**: [ADR-011 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-011-water-leak.md)

## Context

This plugin wraps the `water-leak` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, persistent hiss + drip detector with two-stage gating, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/water-leak` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host.

## Plugin invocation

- `/water-leak` — install if needed, start, tail logs
- `/water-leak --once` — one-shot via `/console` with `--once`
- `/water-leak --console "..."` — pass arbitrary args
- `/water-leak --stop` — stop the cog on the seed
- `/water-leak --logs` — recent stdout/stderr

## RuView mode

None.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/water-leak/`
