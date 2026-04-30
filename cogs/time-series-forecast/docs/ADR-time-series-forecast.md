# ADR: Time Series Forecast as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `time-series-forecast`
**Category**: ai

## Context

Several downstream cogs are useful only if they can ask "where is this signal heading in the next 5/30/60 minutes?" — temperature drift, soil moisture, queue length. ARIMA and Prophet are the textbook answers but pull in heavy linear-algebra dependencies. Holt-Winters triple exponential smoothing is the right size for the Pi Zero 2 W: three running scalars (level, trend, seasonal index) per signal, closed-form updates, immediate forecasts at any horizon, and roughly twelve years of empirical evidence that it works for sub-daily seasonality.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog tracks up to 16 named signals from the `cog-sensor-sources` stream. Each signal runs additive Holt-Winters with auto-tuned α, β, γ via grid search on a 7-day rolling window every 6 hours. Default seasonality length is 24 (one period per hour over a day) but configurable per signal.

State per signal: `(level: f32, trend: f32, season_idx: [f32; L], rmse_24h: f32)`. State machine: **bootstrapping (first L+1 samples) → forecasting (publishing horizons 1, 5, 30, 60 min) → recalibrating (γ-update on full season completion)**.

Output JSON includes the forecast point, a 95 % prediction interval derived from the 24-hour residual RMSE, and the season index so other cogs can detect "sudden mid-cycle deviation."

## Consequences

### Positive
- Closed-form O(1) per-sample update, instant forecasts at any horizon.
- Three scalars + a season array per signal — 16 signals fit easily under 4 KB of state.
- Prediction intervals from residual RMSE are calibrated enough to be actionable.

### Negative
- Single-seasonality assumption misses signals with weekly + daily compound cycles.
- Sudden regime changes cause overshoot for ~one season before α adapts.

### Neutral
- Additive model assumed; multiplicative is opt-in via `--multiplicative` for proportional-noise signals.

## Alternatives considered
- **ARIMA**: rejected — fitting cost and dependency weight.
- **Prophet**: rejected — Python-only, not viable on the seed.

## Plugin invocation
- `/time-series-forecast`
- `/time-series-forecast --once`
- `/time-series-forecast --console "forecast <signal> 30m"`
- `/time-series-forecast --stop`
- `/time-series-forecast --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/time-series-forecast/`
- ADR-001
- `anomaly-attractor`, `pattern-sequence`
