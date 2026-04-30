# plugins/ — 103 cog plugins

One Claude Code plugin per cog in the
[cognitum-one/cogs](https://github.com/cognitum-one/cogs) registry.

Each plugin is a thin wrapper that adds a slash command to your Claude
Code session. The cog itself runs natively as an armhf binary on a paired
[Cognitum Seed](https://cognitum.one) device — it does **not** run inside
your Claude Code session.

## What's in each `<cog-id>/` directory

```
<cog-id>/
├── .claude-plugin/plugin.json   ← manifest (name, version, author, keywords)
├── commands/<cog-id>.md         ← the /<cog-id> slash command body
└── docs/ADR-<cog-id>.md         ← per-cog architecture decision record
```

## Slash command surface (uniform across all 103)

When you install `cog-<cog-id>`, you get one slash command `/<cog-id>` that
wraps the seed's cog management endpoints. The flag pattern is identical
for every cog:

```
/<cog-id>                    install if needed, start on seed, tail logs
/<cog-id> --once             one-shot console execution with --once
/<cog-id> --console "..."    pass arbitrary args (--ruview-mode, --interval N, ...)
/<cog-id> --stop             stop the cog on the seed
/<cog-id> --logs             tail recent stdout/stderr from the seed
```

For example, after `/plugin install cog-fall-detect`:

```
/cog-fall-detect:fall-detect              # install + start + tail
/cog-fall-detect:fall-detect --once       # quick one-shot test
/cog-fall-detect:fall-detect --console "--ruview-mode --impact-threshold 5"
```

## What runs where

```
Claude Code session
  ▼  user: /cog-fall-detect:fall-detect
  ▼
Plugin slash command (commands/fall-detect.md body)
  ▼  Claude reads the markdown body
  ▼  → Bash: curl http://169.254.42.1/api/v1/apps/install -d '{"id":"fall-detect"}'
  ▼  → Bash: curl http://169.254.42.1/api/v1/apps/fall-detect/start
  ▼  → Bash: curl http://169.254.42.1/api/v1/apps/fall-detect/logs
  ▼
Seed agent (Pi-class device, firmware 0.21.x)
  ▼  Downloads cog-fall-detect-arm from gs://cognitum-apps/cogs/arm/
  ▼  Spawns the binary with config-derived argv
  ▼
Cog process on the seed
  ▼  Reads ESP32 sensor stream via cog-sensor-sources
  ▼  Runs detection state machine
  ▼  Writes JSON to output.log + ingests vector to RuVector store
```

The plugin contains no detection code. The cog binary on the seed contains
the detection logic. The plugin just wires the two together via REST.

## Categories (counts)

| Category | Count | Examples |
|----------|-------|----------|
| Health | 14 | sleep-apnea, cardiac-arrhythmia, fall-detect, baby-cry, snore-monitor, cough-detect |
| Security | 14 | intrusion, weapon-detect, glass-break, gunshot-detect, prompt-shield, perimeter-breach |
| AI / ML | 14 | flash-attention, micro-hnsw, rag-local, ewc-lifelong, federated-learning, time-series-forecast |
| Research | 12 | quantum-coherence, time-crystal, hyperbolic-space, sparse-recovery, optimal-transport, temporal-logic |
| Building | 11 | hvac-presence, lighting-zones, water-leak, smoke-fire, frost-warning, beehive-monitor |
| Swarm | 11 | swarm-mesh-manager, swarm-consensus, swarm-distributed-store, swarm-witness-federation |
| Retail | 7 | customer-flow, queue-length, dwell-heatmap, package-detect, parking-occupancy |
| Industrial | 7 | clean-room, confined-space, forklift-proximity, ppe-compliance, slip-fall-zone, predictive-maintenance |
| Developer | 7 | adversarial, anomaly-attractor, anomaly-detect, audit-logger, behavioral-profiler |
| Signal | 6 | breathing-sync, dream-stage, gesture, music-conductor, sound-classifier |

## RuView mode

A subset of cogs honor the ESP32 WiFi-CSI feature stream provided by
[ruvnet/RuView](https://github.com/ruvnet/RuView):

- **Optional** `--ruview-mode` — without the flag the cog uses raw amplitude
  features; with it, CSI features (head-height proxy, motion-drop, gait)
  reinforce the detector. Cogs: `fall-detect`, `gunshot-detect`,
  `slip-fall-zone`, `smoke-fire`, `snore-monitor`, `glass-break`.
- **Required** — cog uses CSI subcarrier-amplitude shifts as its primary
  signal and refuses to run without it. Cogs: `package-detect`,
  `ppe-compliance`, `parking-occupancy`.
- **None** — cog ignores CSI specifics entirely (audio / vibration /
  temperature). All other cogs.

Per-cog details are in each cog's `docs/ADR-<cog-id>.md`. The full
matrix lives at
[cogs/docs/adrs/RUVIEW-CAPABILITY-MATRIX.md](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/RUVIEW-CAPABILITY-MATRIX.md).

## Per-cog config

A cog's slash command can pass any of its `cog.toml`-declared knobs via
`--console "<args>"`. The seed's `/start` endpoint also auto-translates
`config.json` (set via `PUT /api/v1/apps/<id>/config`) into argv for
long-running cogs — so for example, setting `{"ruview_mode": true}` via
the seed's config endpoint is equivalent to passing `--ruview-mode` once.

## Test a single plugin without registering the marketplace

```bash
git clone https://github.com/cognitum-one/cognitum-claude-plugin
claude --plugin-dir cognitum-claude-plugin/plugins/fall-detect

# In the session:
/cog-fall-detect:fall-detect --logs
```

You can stack multiple `--plugin-dir` flags to test several cogs together.

## Adding a new cog plugin

If you ship a new cog to `cognitum-one/cogs`, regenerate the plugin entry
with the script in the seed repo:

```bash
python3 seed/scripts/generate-cog-plugins.py \
    --registry app-registry.json \
    --plugin-repo /tmp/cognitum-claude-plugin-clone \
    --marketplace-out /tmp/marketplace.json
```

This reads the GCS app-registry.json, emits one plugin per cog (with an
auto-generated `commands/<cog-id>.md` and a placeholder
`docs/ADR-<cog-id>.md`), and rebuilds the marketplace.json with the right
`source.path` entries. Hand-edit ADRs for new cogs to capture
domain-specific signal-processing details.

## Related

- [Top-level README](../README.md) — covers the 3 platform plugins
  (cognitum-mcp, cognitum-seed, cog-dev) and how the marketplace fits
  together
- [cognitum-one/cogs](https://github.com/cognitum-one/cogs) — Rust source
  for all 103 cog binaries plus per-cog ADRs in `docs/adrs/`
- [cognitum.one/sdks/claude-code](https://cognitum.one/sdks/claude-code) —
  user-facing usage guide
- [cognitum.one/marketplace.json](https://cognitum.one/marketplace.json) —
  the live marketplace manifest with all 106 plugin entries

## License

Proprietary. See individual `<cog-id>/.claude-plugin/plugin.json` for
per-plugin terms.
