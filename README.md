# cognitum-claude-plugin

Claude Code plugin marketplace for the Cognitum platform — **106 plugins**:

- **3 platform plugins**: cloud MCP, local seed device, cog-dev workflow
- **103 cog plugins**: one Claude Code plugin per cog in the
  [cognitum-one/cogs](https://github.com/cognitum-one/cogs) registry

## Install the marketplace

```bash
/plugin marketplace add https://cognitum.one/marketplace.json
```

That makes all 106 plugins available. Pick the ones relevant to your
deployment.

## Repo layout

```
cognitum-claude-plugin/
├── README.md                    ← you are here
├── .claude-plugin/plugin.json   ← top-level cognitum-mcp plugin
├── .mcp.json                    ← cognitum-mcp MCP server config
├── cognitum-seed/               ← cognitum-seed plugin (114 MCP tools, 8 commands)
├── cog-dev/                     ← cog-dev workflow plugin
└── plugins/                     ← 103 cog plugins, one per directory
    ├── fall-detect/
    │   ├── .claude-plugin/plugin.json
    │   ├── commands/fall-detect.md
    │   └── docs/ADR-fall-detect.md
    ├── cough-detect/
    ├── glass-break/
    ... (100 more)
```

## The 3 platform plugins

| Plugin | Install | What it does |
|--------|---------|--------------|
| `cognitum-mcp` | `/plugin install cognitum-mcp` | Adds the cloud MCP server at https://cognitum.one/mcpSse with 7 tools (health_check, catalog_browse, lead_subscribe, order_status, contact_send, security_verify, fleet_status, docs_search). |
| `cognitum-seed` | `/plugin install cognitum-seed` | Local seed device plugin — 114 MCP tools, 8 slash commands (`/seed-status`, `/seed-pair`, `/seed-wifi`, `/seed-ota`, `/seed-thermal`, `/seed-store`, `/seed-snapshot`, `/seed-cluster`), guided installer agent, troubleshooting skill. Talks to a Seed at `https://169.254.42.1:8443` (LAN) or `http://169.254.42.1` (USB-C). |
| `cog-dev` | `/plugin install cog-dev` | Cog development workflow — 5 slash commands (`/cog-new`, `/cog-test`, `/cog-deploy`, `/cog-trace`, `/cog-config`), 5 skills, 4 subagents. |

## The 103 cog plugins

Each cog in the [cognitum-one/cogs](https://github.com/cognitum-one/cogs)
registry has a corresponding Claude Code plugin under `plugins/<cog-id>/`.
Installing a cog plugin gives you a slash command that wraps the seed's
cog management endpoints:

```
/<cog-id>                   # install the cog on a paired seed if needed, start, tail logs
/<cog-id> --once            # one-shot console execution with --once
/<cog-id> --console "..."   # pass arbitrary args (e.g., --ruview-mode, --interval N)
/<cog-id> --stop            # stop the cog on the seed
/<cog-id> --logs            # tail recent stdout/stderr from the seed
```

The cog itself runs natively as an armhf binary on the Pi-class seed device,
not in your Claude Code session. Each plugin is just the slash-command +
documentation surface.

### By category

| Category | Count | Examples |
|----------|-------|----------|
| Health | 14 | sleep-apnea, cardiac-arrhythmia, fall-detect, baby-cry, snore-monitor, cough-detect |
| Security | 14 | intrusion, weapon-detect, glass-break, gunshot-detect, prompt-shield, audit-logger |
| AI / ML | 14 | flash-attention, micro-hnsw, rag-local, ewc-lifelong, federated-learning, time-series-forecast |
| Research | 12 | quantum-coherence, time-crystal, hyperbolic-space, sparse-recovery, optimal-transport |
| Building | 11 | hvac-presence, lighting-zones, water-leak, smoke-fire, frost-warning, beehive-monitor |
| Swarm | 11 | swarm-mesh-manager, swarm-consensus, swarm-distributed-store, swarm-witness-federation |
| Retail | 7 | customer-flow, queue-length, dwell-heatmap, package-detect, parking-occupancy |
| Industrial | 7 | clean-room, confined-space, forklift-proximity, ppe-compliance, slip-fall-zone, predictive-maintenance |
| Developer | 7 | adversarial, anomaly-attractor, anomaly-detect, audit-logger, behavioral-profiler |
| Signal | 6 | breathing-sync, dream-stage, gesture, music-conductor, sound-classifier |

### Optional ruvnet/ruview integration

A subset of cogs honor the ESP32 WiFi-CSI feature stream provided by
[ruvnet/RuView](https://github.com/ruvnet/RuView):

- **Optional** `--ruview-mode` flag: `fall-detect`, `gunshot-detect`,
  `slip-fall-zone`, `smoke-fire`, `snore-monitor`, `glass-break`. Without
  the flag, the cog runs on raw amplitude features. With the flag, CSI
  features (head-height proxy, motion-drop, pose) reinforce the detector.
- **Required**: `package-detect`, `ppe-compliance`, `parking-occupancy`.
  These cogs use CSI subcarrier-amplitude shifts as their primary signal
  and refuse to run without it.
- **None**: every other cog ignores CSI specifics.

See `plugins/<cog-id>/docs/ADR-<cog-id>.md` per cog for details, or the
[capability matrix](https://github.com/cognitum-one/cogs/blob/main/docs/adrs/RUVIEW-CAPABILITY-MATRIX.md)
in the cogs repo.

## Per-plugin documentation

Each cog plugin ships with:

- `.claude-plugin/plugin.json` — manifest (name, description, version, author, keywords)
- `commands/<id>.md` — the slash command's frontmatter + body (what Claude executes when invoked)
- `docs/ADR-<id>.md` — Architecture Decision Record explaining the cog's domain, signal-processing approach, alternatives considered, and resource budget

The cog *binaries* (armhf ELF for Pi Zero 2 W) live in the cogs repo's
GCS distribution, not here:
```
https://storage.googleapis.com/cognitum-apps/cogs/arm/cog-<id>-arm
```

The seed's `POST /api/v1/apps/install` endpoint downloads them on first
use. The plugin commands wrap that flow.

## Quick test — load a single plugin without installing

You can exercise a plugin from a local checkout without registering the
marketplace:

```bash
git clone https://github.com/cognitum-one/cognitum-claude-plugin
claude --plugin-dir cognitum-claude-plugin/plugins/fall-detect

# now in the session:
/cog-fall-detect:fall-detect --logs
```

That installs `fall-detect` on a paired seed (downloading the armhf binary
from GCS) and tails its log output.

## Repos

- **This repo** (plugin manifests + commands + ADRs): https://github.com/cognitum-one/cognitum-claude-plugin
- **Cog source** (Rust crates that compile to the armhf binaries): https://github.com/cognitum-one/cogs
- **Marketplace JSON** (served from cognitum.one): https://cognitum.one/marketplace.json
- **Cog binary distribution** (GCS): `gs://cognitum-apps/cogs/arm/`
- **Cog registry** (GCS): `gs://cognitum-apps/app-registry.json`

## License

Proprietary. See individual plugin manifests for per-plugin terms.
