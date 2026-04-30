---
name: cog-new
description: Scaffold a new cog from a sensor source and an output spec. Spawns the cog-architect subagent to produce Cargo.toml, cog.toml, and src/main.rs that consume cog-sensor-sources per ADR-091.
arguments:
  - name: id
    description: Cog id (kebab-case; becomes the directory name and binary suffix)
    required: true
---

You are starting a new Cognitum cog. The user just ran `/cog-new {id}` in Claude Code.

# Steps

1. Load the `cog-development` skill for cog.toml schema, sensor-source conventions, and target structure.
2. Spawn the `cog-architect` subagent. Give it:
   - The cog id: `{id}`
   - A request to ask the user 3-4 clarifying questions: sensor source (auto / seed-stream / esp32-uart / esp32-udp), output shape (numeric? JSON? metric stream?), target latency budget, and category (health / security / industrial / etc.).
3. Wait for the architect's draft. It must include:
   - `external/cogs/src/cogs/{id}/Cargo.toml` (binary name `cog-{id}`, depends on `cog-sensor-sources`, `serde`, `serde_json`)
   - `external/cogs/src/cogs/{id}/cog.toml` (name, description, category, [config] schema, sensor source default)
   - `external/cogs/src/cogs/{id}/src/main.rs` (calls `cog_sensor_sources::fetch_sensors()`, processes, emits JSON stream to stdout)
4. Run `cargo check -p cog-{id}` from `external/cogs/`. If it fails, ask the architect to fix.
5. Report success with:
   - Path to scaffolded files
   - One-line usage hint: `/cog-test {id}` to validate, then `/cog-deploy {id}` to ship.

# Constraints

- Stay within `external/cogs/src/cogs/`. Don't touch the seed firmware.
- Use `cog-sensor-sources` — never re-implement UDP/UART parsing per cog (ADR-091).
- Keep the cog binary `<5 MB`. Use `[profile.release]` settings from existing cogs as the template.
- One sensor source per cog. Defaulting to `auto` is the right answer for most.

# Failure modes to flag

- `cog-{id}` already exists — ask the user whether to overwrite.
- Cog id not kebab-case — refuse and explain.
- User asks for a sensor type that doesn't fit through `cog-sensor-sources` (e.g. raw I2C) — the architect should flag this; offer to extend the shared crate instead of building a one-off.
