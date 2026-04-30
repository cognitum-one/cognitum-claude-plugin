# ADR: Seizure Detection as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `seizure-detect`
**Category**: health

## Context

Tonic-clonic seizures have a specific motion signature: rhythmic,
high-amplitude convulsions in the 2–6 Hz band, sustained for tens of
seconds, with no voluntary postural reset. SUDEP (sudden unexpected
death in epilepsy) is the worst outcome epilepsy patients face, and it
is strongly correlated with **unwitnessed nocturnal seizures**. A
silent bedroom seed that fires a notification within seconds of the
convulsion onset is an unambiguous quality-of-life win for the family.

Target deployment is residential (pediatric and adult epilepsy
households) and clinical (epilepsy monitoring step-down rooms where
budget doesn't stretch to per-bed video EEG). The cog needs to be
**fast** — a 30s alert is useful, a 5 min alert is not — so the
sampling interval is the most aggressive of the health cogs (default 3s).

Approach: spectral energy in the 2–6 Hz seizure band, sustained above
a z-score threshold for ≥10 s, with simultaneous broadband motion
amplitude well above quiet baseline. Both gates must hold to fire
`SEIZURE_DETECTED`.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 3s, range 1–60s).
Per frame it computes seizure-band energy and broadband variance; both
above their thresholds across a 10s rolling window emits the alert and
holds it until the convulsion subsides.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/seizure-detect` that wraps the seed's cog management
endpoints (install, start, console, stop, logs).

## Consequences

### Positive
- Catches the failure mode that kills epilepsy patients (nocturnal
  unwitnessed convulsions) without requiring a worn device the
  patient may forget or refuse.
- Sub-30s alert latency at default 3s sampling — fast enough for a
  caregiver in the next room to intervene during the postictal phase.
- The dual-gate design (band energy AND broadband variance) suppresses
  the obvious false positives: rolling over in bed, sleep myoclonus,
  partner restlessness.

### Negative
- Cannot detect non-convulsive seizures (absence, focal-aware) — they
  have no motion signature. Out of scope; that needs EEG.
- High-confidence alerts still need human verification before calling
  EMS; the cog is decision support, not autonomous medical action.

### Neutral
- The 10s sustain requirement trades onset latency for false-positive
  suppression; tuning is per-deployment (kids vs. adults differ).

## Alternatives considered

- **Wearable seizure detectors (Embrace, etc.).** Rejected for v1: not
  worn during sleep by a meaningful fraction of patients, which is
  exactly when SUDEP risk is highest.
- **Camera + ML.** Rejected: privacy hostile in a child's bedroom and
  busts the Pi Zero 2 W resource budget; bandpass + variance gets
  most of the yield.

## Plugin invocation

- `/seizure-detect` — install if needed, start, tail logs
- `/seizure-detect --once` — one-shot console execution with `--once`
- `/seizure-detect --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 1`)
- `/seizure-detect --stop` — stop the running cog
- `/seizure-detect --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/seizure-detect/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related cogs: `fall-detect` (shares the bandpass + variance state
  machine pattern), `respiratory-distress` (postictal apnea is a known
  SUDEP precursor — pairing the two raises confidence)
