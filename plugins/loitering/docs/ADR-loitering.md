# ADR: Loitering Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `loitering`
**Category**: security

## Context

A motion event of "someone is in the alley" is mostly noise — people
walk past alleys all day. The actionable variant is "someone has been
in the alley **for two minutes without leaving**." Loitering detection
is dwell-time on top of motion presence, and it is one of the
highest-yield, lowest-cost cogs you can run on a seed pointed at a
back lot, ATM vestibule, or restricted hallway.

Target deployment is commercial / municipal: ATM enclosures,
construction-site after-hours, retail back-of-house, parking
structures, and residential property edges. The signal is simple:
**continuous presence (motion above quiet baseline) sustained beyond
`loiter_time` seconds without a corresponding departure**.

Approach: per frame, classify present / absent against the running
quiet baseline. Increment a dwell timer while present; reset on
absent. Dwell crossing `loiter_time` (default 120s, range 10–3600s)
emits `LOITERING_DETECTED`.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 10s). It
maintains a single integer dwell counter and a present/absent
classifier; both fit comfortably in the 3 KB description footprint
that makes this the smallest of the security cogs.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/loitering` that wraps the seed's cog management endpoints
(install, start, console, stop, logs).

## Consequences

### Positive
- Cuts presence-event noise by orders of magnitude — a single
  walk-through doesn't fire, only sustained presence does.
- Trivial to deploy and explain to non-technical operators ("alert me
  if someone stays for more than 2 minutes"); the threshold maps
  directly onto a stopwatch.
- The smallest cog in the catalog by binary size — easy to stack
  alongside `intrusion` and `perimeter-breach` on the same seed.

### Negative
- Cannot distinguish a customer waiting legitimately from a
  prospective intruder; deployment context (after-hours config,
  geofencing) does that work.
- A static object in the sensor view (a parked car) can produce a
  spurious sustained-presence reading until the baseline re-learns.

### Neutral
- The 10s default interval is comfortable for human-scale dwell
  thresholds; tighter intervals only help if you're chaining this
  cog with a fast tracker like `tailgating`.

## Alternatives considered

- **Camera + ML person tracking.** Rejected: cost, privacy, and
  the Pi Zero 2 W resource budget. The dwell-counter approach
  catches the high-yield cases at near-zero cost.
- **Hardcoded dwell threshold.** Rejected: a 2-minute threshold is
  right for an ATM and absurd for a maintenance crew break room;
  the deployment must own the threshold.

## Plugin invocation

- `/loitering` — install if needed, start, tail logs
- `/loitering --once` — one-shot console execution with `--once`
- `/loitering --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--loiter-time 300`)
- `/loitering --stop` — stop the running cog
- `/loitering --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/loitering/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related security cogs: `intrusion` (single-event presence detector
  this dwell-cog enriches), `perimeter-breach` (zone-aware sibling
  that pairs naturally with loitering for "where did they linger?")
