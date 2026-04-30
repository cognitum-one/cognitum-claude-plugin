# ADR: Respiratory Distress as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `respiratory-distress`
**Category**: health

## Context

Respiratory rate is the most sensitive vital sign for early
deterioration — it usually rises hours before blood pressure or heart
rate cross alerting thresholds. Hospitals know this, but home and
assisted-living deployments rarely measure it because traditional
monitors are intrusive.

Target deployment is residential (post-discharge recovery, COPD, asthma,
RSV in infants) and clinical (low-acuity step-down rooms). The signal of
interest is **tachypnea (sustained breathing rate above ~30 BPM)** plus
**morphology changes** like sustained shallow or labored breathing,
which show up as a drop in the breathing-band envelope amplitude
relative to baseline.

Approach: bandpass to ~0.1–0.5 Hz, count zero-crossings to estimate BPM,
and z-score the envelope amplitude against the running baseline. Acute
deviations fire `RESPIRATORY_DISTRESS`.

## Decision

The cog runs as a standalone armhf binary on the seed, reading
`cog-sensor-sources` every `interval` seconds (default 5s). It maintains
two rolling stats: instantaneous BPM (1 min window) and amplitude
z-score (5 min baseline). Either metric crossing its threshold fires
the structured alert; both crossing simultaneously raises confidence.

When packaged as a Claude Code plugin (this directory), it adds slash
command `/respiratory-distress` that wraps the seed's cog management
endpoints (install, start, console, stop, logs).

## Consequences

### Positive
- Contactless and continuous — no chest band, no nasal cannula. Picks
  up infant respiratory illness without waking the child.
- Rate **and** morphology gives two independent failure modes; an
  agitated kid (rate up, amplitude up) looks different from a tiring
  COPD patient (rate up, amplitude down).
- The 5s default interval is fast enough to alert before a clinician
  walking by would notice.

### Negative
- Misses brief breath-holding episodes shorter than the 1 min BPM
  window; pair with `sleep-apnea` for sub-minute events.
- The amplitude proxy is sensitive to the subject's distance from the
  seed; movement around the room re-baselines spuriously.

### Neutral
- Tachypnea threshold defaults to ~30 BPM (clinical adult cutoff);
  pediatric deployments need configuration.

## Alternatives considered

- **SpO2 finger clip.** Rejected for v1: requires worn hardware. SpO2
  is a complementary signal and a strong v2 mesh input.
- **Single-metric alert (rate-only).** Rejected: misses the labored-but-
  slow failure mode where amplitude collapses before rate climbs.

## Plugin invocation

- `/respiratory-distress` — install if needed, start, tail logs
- `/respiratory-distress --once` — one-shot console execution with `--once`
- `/respiratory-distress --console "<args>"` — pass arbitrary args (e.g. `--ruview-mode`, `--interval 3`)
- `/respiratory-distress --stop` — stop the running cog
- `/respiratory-distress --logs` — print recent stdout/stderr

## Resource budget

- Binary: ~ 400-460 KB stripped armhf (within ADR-001 budget for cog-as-plugin).
- RAM: < 2 MB resident.
- CPU: < 5% of one Pi Zero 2 W core at default sampling rate.

## See also

- Cog source: `cognitum-one/cogs:src/cogs/respiratory-distress/`
- Foundational architecture: ADR-001 (cogs as plugins)
- Related health cogs: `sleep-apnea` (chronic / sub-minute version of
  the same signal), `vital-trend` (provides the BPM baseline this cog
  alerts against)
