# ADR: Parking Occupancy as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `parking-occupancy`
**Category**: retail
**Canonical cog ADR**: [ADR-016 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-016-parking-occupancy.md)

## Context

This plugin wraps the `parking-occupancy` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, per-zone CSI subcarrier-amplitude shift detector, utilization and churn metrics, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/parking-occupancy` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host with an attached ESP32 ruview feed sited to cover the parking zones.

## Plugin invocation

- `/parking-occupancy` — install if needed, start, tail logs
- `/parking-occupancy --once` — one-shot via `/console` with `--once`
- `/parking-occupancy --console "..."` — pass arbitrary args
- `/parking-occupancy --stop` — stop the cog on the seed
- `/parking-occupancy --logs` — recent stdout/stderr

## RuView mode

Required. Cog assumes the ESP32 ruview firmware feeds CSI subcarrier amplitudes through the standard feature stream.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/parking-occupancy/`
