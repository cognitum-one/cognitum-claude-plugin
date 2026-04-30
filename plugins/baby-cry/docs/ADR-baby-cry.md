# ADR: Baby Cry Detection as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `baby-cry`
**Category**: health

The canonical ADR for this cog is in `cognitum-one/cogs/docs/adrs/ADR-004-baby-cry.md`.
That document covers the sustained mid-band energy detector, the
audio-only (no camera) design constraint, the nursery / infant
monitoring use case, and the alternatives considered. **This plugin
wraps that cog as a Claude Code slash command** — no behavioral changes.
See the canonical ADR for problem framing and design rationale.

## Plugin invocation

- `/baby-cry` install, start, tail
- `/baby-cry --once`
- `/baby-cry --console "--interval 1 --cry-z 2.5 --cry-min-secs 2 --cooldown 15"`
- `/baby-cry --stop`
- `/baby-cry --logs`

## Resource budget

- Binary: 400-460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also

- Canonical ADR: `cognitum-one/cogs/docs/adrs/ADR-004-baby-cry.md`
- `cognitum-one/cogs:src/cogs/baby-cry/`
- ADR-001 (cogs-as-plugins architecture)
- `cough-detect`, `fall-detect`
