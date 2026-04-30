# ADR: Air Quality Monitor as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `air-quality-index`
**Category**: health

## Context

When a CO2 / VOC / PM2.5 sensor is wired to the ESP32 (SCD4x or BME680 over I2C, plus an optional PM sensor on UART), the raw readings flow into the seed as additional channels on `cog-sensor-sources`. Users want a single AQI number, not three sensor streams. Different agencies define AQI differently; we follow the EPA breakpoints, but cleanly: one cog, one number, with the inputs visible.

If no AQ sensors are attached, the cog should detect that and refuse to invent data.

## Decision

`air-quality-index` reads CO2 (ppm), VOC index, and PM2.5 (µg/m³) channels from the feature stream every 30 s (default), maps each to a sub-index using EPA breakpoints, and reports the worst sub-index as the overall AQI plus the dominant pollutant. On startup, if any required channel is absent for 60 s, the cog reports `aqi_unavailable` with the missing inputs listed — never zero-filled.

## Consequences

### Positive
- 8 KB binary; trivial CPU.
- Honest about missing sensors instead of silently producing bogus AQI.
- EPA breakpoints are well-documented and auditable.

### Negative
- Locked to the EPA scale; users in EU/CN regions may want a different scale (open issue).

### Neutral
- VOC index from BME680 is a relative scale; we map it conservatively.

## Alternatives considered

- **Average all sub-indices.** Rejected: hides spikes — a brief PM event matters even if CO2 is fine.
- **Bundle into `health-monitor`.** Rejected: AQ is environmental, not biometric, and runs without a person present.

## Plugin invocation
- `/air-quality-index` install, start, tail
- `/air-quality-index --once`
- `/air-quality-index --console "--interval 30"`
- `/air-quality-index --stop`
- `/air-quality-index --logs`

## Resource budget
- Binary: ~410 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/air-quality-index/` | ADR-001 | health-monitor | hvac-presence
