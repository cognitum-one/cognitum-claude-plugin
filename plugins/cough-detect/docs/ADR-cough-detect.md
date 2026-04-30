# ADR: Cough Detection as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `cough-detect`
**Category**: health

The canonical ADR for this cog is in `cognitum-one/cogs/docs/adrs/ADR-003-cough-detect.md`.
That document covers the acoustic transient + spectral detector, the
30-second cluster aggregation window, the early-warning use case for
respiratory illness, and the alternatives considered. **This plugin
wraps that cog as a Claude Code slash command** — no behavioral changes.
See the canonical ADR for problem framing and design rationale.

## Plugin invocation

- `/cough-detect` install, start, tail
- `/cough-detect --once`
- `/cough-detect --console "--interval 1 --transient-z 3.0 --cluster-window 30 --alert-count 3"`
- `/cough-detect --stop`
- `/cough-detect --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- Canonical ADR: `cognitum-one/cogs/docs/adrs/ADR-003-cough-detect.md`
- `cognitum-one/cogs:src/cogs/cough-detect/`
- ADR-001 (cogs-as-plugins architecture)
- `baby-cry`, `fall-detect`
