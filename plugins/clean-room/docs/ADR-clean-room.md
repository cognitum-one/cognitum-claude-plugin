# ADR: Clean Room as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `clean-room`
**Category**: industrial

## Context

ISO 14644 cleanroom classification depends on particle generation, which
is dominated by people: each additional body adds a known particle load
per cubic foot per minute. Pharma, semiconductor, and biotech sites
enforce per-room headcount caps but typically rely on badge readers at
entry — which fail open if a door is propped or two enter on one badge.
A passive headcount oracle that does not require badging gives the QA
team an independent compliance signal.

A useful headcount monitor on a Pi Zero 2 W must:

1. Estimate occupants in the room rather than tracking individuals.
2. Trigger alert immediately when count exceeds the configured
   `--max-occupancy`.
3. Operate without cameras (regulated areas often forbid them).

## Decision

`clean-room` clusters `cog-sensor-sources` activity across a coarse 3 s
window into a count estimate using motion variance and CSI multipath
diversity. State machine: `vacant → within-cap → AT-CAP → OVER-CAP`.
`OVER-CAP` fires a local buzzer and a mesh broadcast to a hallway
display. Output: `count`, `state`, `over_seconds`, `last_change_ts`.

As a Claude Code plugin, `/clean-room` returns live count, today's
over-cap minutes (a compliance KPI), and supports acknowledging an alert
once the room is brought back within cap.

## Consequences

### Positive
- Camera-free — works in regulated suites where optical install is
  prohibited.
- Independent of badge infrastructure; backstops it rather than
  replacing it.
- Over-cap minute log is directly auditable for compliance review.

### Negative
- Counts are estimates with ±1 person error in steady state — fine for
  cap enforcement, not for precise occupancy billing.

### Neutral
- Per-room calibration on install determines how the variance signal
  maps to count.

## Alternatives considered

- **Badge turnstile + interlock.** Complementary, not replacement —
  fails open on tailgating and does not detect badged-in-but-left.
- **Ceiling-mount thermal counter.** Rejected on cost and the burden
  of obtaining a regulated-environment install permit.

## Plugin invocation

- `/clean-room` — install if needed, start, tail logs
- `/clean-room --once` — current count snapshot
- `/clean-room --console "<args>"` — arbitrary args
- `/clean-room --stop` — stop
- `/clean-room --logs` — output

## Resource budget

- Binary: 400-460 KB stripped armhf.
- RAM: < 2 MB.
- CPU: < 5% of one core.

## See also

- Source: `cognitum-one/cogs:src/cogs/clean-room/`
- ADR-001 (foundational).
- `confined-space`, `customer-flow`.
