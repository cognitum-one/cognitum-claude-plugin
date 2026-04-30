# ADR: Neural Trader as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `neural-trader`
**Category**: ai

## Context

A subset of cognitum users want the seed to monitor a price feed (crypto tickers, FX) and surface pattern signals — momentum breaks, mean-reversion candidates, volume divergences — without running a full trading stack. Crucially: **this cog does not place trades**. It is a pattern surfacer that the user (or an external trading system) can act on.

The Pi Zero 2 W cannot run a real model. What it can do is pull a tick stream over the existing HTTP layer and run cheap classical indicators (EMA crossover, RSI, Bollinger band extremes) plus a small fixed-weight MLP — really a learned linear combination of those indicators — at 60 s intervals.

## Decision

`neural-trader` polls a configured price endpoint on `--interval` (default 60 s), maintains rolling indicator state for the active symbols, and emits `trade_signal` JSON: symbol, direction (`long`/`short`/`flat`), confidence (heuristic), and which indicators agreed. It will refuse to start if `EXECUTION_ENABLED=true` is set anywhere — placement is explicitly out of scope.

## Consequences

### Positive
- Honest scope: pattern detection, not order routing.
- 20 KB binary; the only real cost is HTTP polling.
- Featured cog — useful demo of the "seed as ambient analyst" framing.

### Negative
- Indicators are well-known and the MLP weights are static; alpha decay is real.

### Neutral
- Confidences are heuristic and uncalibrated. Treat them as ranking, not probability.

## Alternatives considered

- **Full trading bot with execution.** Rejected: out of scope for a sensor appliance, and a regulatory minefield.
- **LLM-driven analysis.** Rejected: latency, cost, offline operation requirement.

## Plugin invocation
- `/neural-trader` install, start, tail
- `/neural-trader --once`
- `/neural-trader --console "--interval 60"`
- `/neural-trader --stop`
- `/neural-trader --logs`

## Resource budget
- Binary: ~460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/neural-trader/` | ADR-001 | interference-search (template scoring)
