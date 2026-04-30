# ADR: PPE Compliance as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `ppe-compliance`
**Category**: industrial
**Canonical cog ADR**: [ADR-009 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-009-ppe-compliance.md)

## Context

This plugin wraps the `ppe-compliance` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, cog-composition layer fusing ruview-densepose presence with PPE-camera-cog confirmation, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/ppe-compliance` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host with ruview-densepose and a PPE confirmation cog already running.

## Plugin invocation

- `/ppe-compliance` — install if needed, start, tail logs
- `/ppe-compliance --once` — one-shot via `/console` with `--once`
- `/ppe-compliance --console "..."` — pass arbitrary args
- `/ppe-compliance --stop` — stop the cog on the seed
- `/ppe-compliance --logs` — recent stdout/stderr

## RuView mode

Required. Cog assumes the ESP32 ruview firmware feeds CSI subcarrier amplitudes through the standard feature stream, and that ruview-densepose is producing zone-presence vectors.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/ppe-compliance/`
