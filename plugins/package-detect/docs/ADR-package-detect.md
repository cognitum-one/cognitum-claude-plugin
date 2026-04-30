# ADR: Package Arrival Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `package-detect`
**Category**: retail
**Canonical cog ADR**: [ADR-008 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-008-package-detect.md)

## Context

This plugin wraps the `package-detect` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, sustained CSI-shift detector, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/package-detect` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host with an attached ESP32 ruview feed.

## Plugin invocation

- `/package-detect` — install if needed, start, tail logs
- `/package-detect --once` — one-shot via `/console` with `--once`
- `/package-detect --console "..."` — pass arbitrary args
- `/package-detect --stop` — stop the cog on the seed
- `/package-detect --logs` — recent stdout/stderr

## RuView mode

Required. Cog assumes the ESP32 ruview firmware feeds CSI subcarrier amplitudes through the standard feature stream.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/package-detect/`
