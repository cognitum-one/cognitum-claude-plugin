# ADR: Temporal Logic Guard as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `temporal-logic`
**Category**: ai

## Context

The seed accumulates safety rules an operator wants enforced live: "if smoke detected, fan must turn off within 30 s", "the front door must never be unlocked when no one is home for more than 5 minutes", "alert if the freezer has been open for >2 minutes." These are textbook Metric Temporal Logic (MTL) safety properties. Hand-coding each one as a state machine is tedious and bug-prone; a tiny MTL evaluator handles the whole class declaratively.

A bounded-history monitor for past-time MTL evaluates each rule in O(formula_size) per event without re-scanning the trace, which is what fits on the seed.

## Decision

Standalone armhf binary on Pi Zero 2 W. Rules are written in past-time MTL with operators `[Y]` (yesterday/previous), `[H]` (historically), `[O]` (once-in-past), `[S]` (since), each with metric bounds `[a,b]` in seconds. The cog parses each rule into an evaluation tree, maintains a per-subformula sliding-window state (boolean ring buffer for atomic propositions, pair-counter for binary operators), and at each interval evaluates every rule against the live event timestamp and emits violations.

State machine per rule: **satisfied → pending (deadline armed) → VIOLATION → cooldown**. Rules persist as `rules.toml`; the violation channel publishes a JSON event other cogs can subscribe to.

## Consequences

### Positive
- Declarative rules — operators write `H[0,300]( door_unlocked → presence )`, no Rust code needed.
- O(1) per-event update per rule means dozens of rules cost negligibly.
- Past-time MTL is decidable and total — no diverging evaluations.

### Negative
- Past-time only — cannot express "eventually within 5 min" (future-time) without bounded look-ahead.
- Window memory grows with the largest metric bound; very long bounds (hours) push RAM.

### Neutral
- Rules are operator-authored and trusted; no formal verification pass.

## Alternatives considered
- **Future-time LTL with tableau**: rejected — non-trivial memory and decision delay incompatible with live alerts.
- **Hand-coded state machines per rule**: rejected — duplication, no compositionality.

## Plugin invocation
- `/temporal-logic`
- `/temporal-logic --once`
- `/temporal-logic --console "check rules"`
- `/temporal-logic --stop`
- `/temporal-logic --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/temporal-logic/`
- ADR-001
- `pattern-sequence`, `goap-autonomy`
