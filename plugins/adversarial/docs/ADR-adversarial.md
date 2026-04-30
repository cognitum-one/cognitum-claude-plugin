# ADR: Adversarial Detection as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `adversarial`
**Category**: developer

## Context

A seed deployed in a security-relevant role (intrusion, gunshot, fall-detect) is vulnerable to feature-stream tampering: replayed frames, frozen baselines, white-noise injection to mask events, or constant-value spoofing to hide presence. `coherence` flags broken streams; `adversarial` flags streams that look *too clean* or repeat too perfectly to be real.

The Pi Zero 2 W cannot run heavy anomaly models, but most spoofs leave fingerprints in low-order statistics: zero variance over long windows, exact-equal consecutive frames, periodic repeats with low entropy.

## Decision

`adversarial` runs at 0.1 Hz over `cog-sensor-sources` and applies four cheap detectors: (a) frame-equality runs > 30 s, (b) chi-squared deviation of feature histograms from the trained baseline, (c) auto-correlation peaks suggesting loop replay, (d) entropy floor on the LSB bits. Any detector tripping for two consecutive intervals raises `tamper_suspected` with which detector fired.

## Consequences

### Positive
- Cheap (4 KB) and continuous — runs alongside any other cog.
- Catches the obvious attacks (replay, freeze, constant injection) without ML.
- Detector attribution makes triage tractable.

### Negative
- A motivated attacker recording and re-streaming a long, varied loop will pass.

### Neutral
- Calibration requires a clean baseline window; deployments with already-tampered streams produce useless baselines.

## Alternatives considered

- **GAN-based anomaly detector.** Rejected: model size and CPU budget on Pi Zero 2 W.
- **Cryptographic stream signing on ESP32.** Promising for v2; would belong in firmware, not a cog.

## Plugin invocation
- `/adversarial` install, start, tail
- `/adversarial --once`
- `/adversarial --console "--interval 10"`
- `/adversarial --stop`
- `/adversarial --logs`

## Resource budget
- Binary: ~400 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/adversarial/` | ADR-001 | coherence | intrusion-detect-ml
