# ADR: Predictive Maintenance as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `predictive-maintenance`
**Category**: building
**Canonical cog ADR**: [ADR-015 in cognitum-one/cogs](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-015-predictive-maintenance.md)

## Context

This plugin wraps the `predictive-maintenance` cog as a Claude Code slash command. The cog itself is documented in the canonical ADR linked above (problem statement, vibration harmonic severity scoring, alternatives, resource budget). This document covers only the plugin packaging concerns.

## Decision

Adds slash command `/predictive-maintenance` that calls the seed's cog management endpoints to install (download armhf binary from GCS registry on first use), start, run, and stop the cog. The cog runs natively on the seed (Pi Zero 2 W), not in the Claude Code session. Requires a paired seed reachable at the configured host with an accelerometer/IMU mounted on the rotating equipment under test.

## Plugin invocation

- `/predictive-maintenance` — install if needed, start, tail logs
- `/predictive-maintenance --once` — one-shot via `/console` with `--once`
- `/predictive-maintenance --console "..."` — pass arbitrary args
- `/predictive-maintenance --stop` — stop the cog on the seed
- `/predictive-maintenance --logs` — recent stdout/stderr

## RuView mode

None.

## See also

- Canonical ADR: link above
- Foundational: ADR-001 (cogs as plugins) at https://github.com/cognitum-one/cogs/blob/main/docs/adrs/ADR-001-cogs-as-plugins-architecture.md
- Plugin source: `cognitum-one/cognitum-claude-plugin/cogs/predictive-maintenance/`
