# ADR: Fall Detection as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `fall-detect`
**Category**: health

The canonical ADR for this cog is in `cognitum-one/cogs/docs/adrs/ADR-002-fall-detect.md`.
That document covers the two-stage impact + stillness state machine, the
ambient-feature input contract, the optional RuView/CSI reinforcement
mode, and the alternatives considered. **This plugin wraps that cog as
a Claude Code slash command** — no behavioral changes. See the canonical
ADR for problem framing and design rationale.

## Plugin invocation

- `/fall-detect` install, start, tail
- `/fall-detect --once`
- `/fall-detect --console "--interval 1 --impact-threshold 6.0 --stillness-window 8 --cooldown 30"`
- `/fall-detect --console "--ruview-mode"` (CSI head-height reinforcement)
- `/fall-detect --stop`
- `/fall-detect --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- Canonical ADR: `cognitum-one/cogs/docs/adrs/ADR-002-fall-detect.md`
- `cognitum-one/cogs:src/cogs/fall-detect/`
- ADR-001 (cogs-as-plugins architecture)
- `gait-analysis`, `cough-detect`
