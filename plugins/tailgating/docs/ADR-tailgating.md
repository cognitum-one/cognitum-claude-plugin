# ADR: Tailgating Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `tailgating`
**Category**: security

## Context

Tailgating — a second person slipping through a controlled door
behind a legitimate badge swipe — is the most common physical
breach in commercial-access-control audits, and almost no badge
reader detects it. The detection problem is short and well-defined:
**count distinct human transits in the time window after a badge
event** and compare to expected (1 swipe = 1 person).

Target deployment is commercial / institutional access points:
office lobbies, server-room doors, HIPAA-restricted clinical wings.
The seed lives near the badge reader, watches the doorway feature
stream for the post-swipe window, and counts motion peaks
corresponding to distinct transits.

Approach: a single transit produces one motion-amplitude pulse with
a characteristic duration. Two pulses inside a short window after a
badge event = tailgate. Pulse separation has to be greater than a
minimum dwell (people don't pass through an instant apart) and less
than a max window (after the door auto-closes the count resets).

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 1s — needs to
be fast to resolve adjacent transits). On each badge event from the
upstream access-control integration, it opens a window, counts
transits, and emits `TAILGATE_DETECTED` if the count exceeds 1.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/tailgating` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Closes the highest-volume physical-security gap in commercial
  access control with no badge-reader change.
- Single-pulse-counting is robust and explainable — the alert
  comes with a transit count an auditor can sanity-check.
- Pairs cleanly with the existing badge-event log; the cog only
  fires inside the post-swipe window so false-positive rate is
  bounded by badge frequency.

### Negative
- Cannot identify the tailgater — only flags that one occurred.
  Identification needs camera or a second badge swipe inside the
  zone.
- Requires upstream badge-event integration; without it, the cog has
  no window to watch and effectively reduces to motion counting.

### Neutral
- The transit-window length is the dominant tunable; too short
  misses laggy follow-throughs, too long false-positives on the
  next legitimate person 30s later.

## Alternatives considered

- **Mantrap / turnstile.** Rejected as a replacement — the cog is the
  retrofit answer for buildings whose front doors will never become
  turnstiles.
- **Camera-based people counting.** Rejected for v1: cost, privacy,
  Pi Zero 2 W resource budget. Could be a v2 mesh input where
  cameras already exist.

## Plugin invocation

- `/tailgating` — install if needed, start, tail logs
- `/tailgating --once` — one-shot console execution with `--once`
- `/tailgating --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 1`)
- `/tailgating --stop` — stop the running cog
- `/tailgating --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/tailgating/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related security cogs: `weapon-detect` (companion cog at the same
  doorway — they answer "who came through, and were they carrying?"),
  `perimeter-breach` (the zone-level alert this cog's transit count
  feeds into)
