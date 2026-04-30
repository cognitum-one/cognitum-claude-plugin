# ADR: DTW Gesture Learn as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `dtw-gesture-learn`
**Category**: ai

## Context

Off-the-shelf gesture recognizers ship a fixed alphabet (swipe up, wave, etc.) and require a labelled training set. The seed deployment scenario is the opposite — a single user wants to teach two or three idiosyncratic gestures (a triple-tap on the bedside table, a hand circle in front of the camera) and have them fire commands. The seed has roughly 460 KB of binary budget per cog and no GPU.

Dynamic Time Warping is a strong fit: it compares variable-length sequences without training, tolerates speed differences between the template and the live input, and the Sakoe-Chiba banded variant runs in O(n·w) which is tractable on the Pi Zero 2 W's ARM11.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog consumes the `cog-sensor-sources` 8-feature stream (CSI motion bins + audio envelope) and runs banded DTW (band width 10) against a small library of user-recorded templates stored in `~/.cognitum/dtw-gesture-learn/templates.cbor`.

State machine: **idle → recording (5 s window on `/learn` API) → matching (interval-paced)**. Each match cycle z-normalises the live window, computes DTW distance to every template, and emits the best match if distance is below `match_threshold` (default 0.35 of template length).

## Consequences

### Positive
- No model training, no labelled data — the user demonstrates a gesture once.
- Banded DTW keeps each match under 8 ms for a 50-frame template.
- Template library is human-readable CBOR; transferable between seeds.

### Negative
- DTW does not generalise; a gesture done with the other hand reads as a different template.
- Memory grows linearly with template count (≈2 KB each).

### Neutral
- Threshold tuning is per-deployment; the default suits indoor CSI features.

## Alternatives considered
- **1-NN classifier on hand-crafted features**: rejected — needs a labelled set the user does not have.
- **TinyML CNN on accelerometer**: rejected — wearable required, exceeds RAM budget for on-device training.

## Plugin invocation
- `/dtw-gesture-learn`
- `/dtw-gesture-learn --once`
- `/dtw-gesture-learn --console "learn wave"`
- `/dtw-gesture-learn --stop`
- `/dtw-gesture-learn --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/dtw-gesture-learn/`
- ADR-001
- `pattern-sequence`, `spiking-tracker`
