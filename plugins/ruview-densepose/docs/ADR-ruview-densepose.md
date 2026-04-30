# ADR: RuView WiFi DensePose as Claude Code plugin
**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `ruview-densepose`
**Category**: research

## Context

RuView (ADR-084 / external/ruview) extracts CSI features from an ESP32 WiFi front-end. With enough subcarriers and a still scene, the literature shows you can recover a coarse skeleton — head, shoulders, hips, knees — without a camera. The seed has the front-end; what is missing on the Pi Zero 2 W side is the inference cog.

This is genuinely a research cog. Quality varies wildly with antenna placement, multipath, and how many bodies are in the scene. On a good day it produces a 17-keypoint skeleton at ~1 Hz; on a bad day it produces noise. We are honest about this.

## Decision

`ruview-densepose` subscribes to the extended CSI feature stream (RuView mode of `cog-sensor-sources`), runs a small fixed-weight regressor (distilled offline against synthetic data) at the configured interval (default 1 s), and emits 17 keypoints in a normalized room frame plus a per-keypoint confidence. When confidence median drops below 0.4, output is suppressed and `pose_unreliable` is published instead.

## Consequences

### Positive
- Privacy-preserving body tracking without cameras.
- Featured research cog — concrete demonstration of the WiFi-sensing track.
- Confidence-gated output prevents downstream cogs from acting on hallucinated skeletons.

### Negative
- 50 KB binary is the largest in this batch; still well inside budget.
- Accuracy is environment-dependent and degrades with multiple occupants.

### Neutral
- Calibration to a deployment's antenna geometry takes ~10 minutes of supervised motion.

## Alternatives considered

- **Camera-based MediaPipe pose.** Rejected: privacy, and breaks the "no camera" seed contract.
- **mmWave radar pose.** Rejected: hardware not on the default seed BOM.

## Plugin invocation
- `/ruview-densepose` install, start, tail
- `/ruview-densepose --once`
- `/ruview-densepose --console "--interval 1"`
- `/ruview-densepose --stop`
- `/ruview-densepose --logs`

## Resource budget
- Binary: ~460 KB armhf | RAM: < 2 MB | CPU: < 5%

## See also
- `cognitum-one/cogs:src/cogs/ruview-densepose/` | ADR-001 | ADR-084 (mesh / RuView) | fall-detect (`--ruview-mode`)
