# ADR: Energy Harvester as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `energy-harvester`
**Category**: research

## Context

Off-grid seed deployments (farms, sheds, remote hives) run on solar + battery. The Pi Zero 2 W idles around 0.4 W, which a 5 W panel and a 10 Wh battery handle in clear weather but not in winter or under leaf cover. Without a duty-cycle policy, the seed runs flat at 03:00 and misses the morning event the user actually cared about.

`energy-harvester` is the policy. It reads battery voltage and panel current from an attached INA219 (or similar), forecasts short-term solar yield from a rolling daily profile, and recommends which cogs to suspend.

## Decision

`energy-harvester` runs every 10 s, samples battery state-of-charge and panel power, and updates a state-of-energy estimate. When projected SOC at sunrise drops below `min_dawn_soc` (default 30%), it publishes `power_pressure` with a tiered shed list — research/AI cogs first (`quantum-coherence`, `neural-trader`, `psycho-symbolic`), then non-essential health cogs, then everything but a heartbeat. Cogs subscribe and self-suspend; the harvester does not kill processes.

## Consequences

### Positive
- Seeds survive overnight on a small battery without manual intervention.
- 6 KB binary; near-zero CPU.
- Suspend tiers are configurable per deployment.

### Negative
- Forecast is a 7-day rolling average, so the first week of deployment runs blind and may over-shed or under-shed.

### Neutral
- Only useful if a current-sense IC is wired in; on USB-powered deployments the cog reports `mains_powered` and idles.

## Alternatives considered

- **Hard duty-cycle schedule.** Rejected: doesn't adapt to weather.
- **Cloud-side power policy.** Rejected: offline operation; also the cloud doesn't know your panel orientation.

## Plugin invocation
- `/energy-harvester` install, start, tail
- `/energy-harvester --once`
- `/energy-harvester --console "--interval 10"`
- `/energy-harvester --stop`
- `/energy-harvester --logs`

## Resource budget
- Binary: ~400 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/energy-harvester/` | ADR-001 | energy-audit | self-healing-mesh
