---
name: cog-test
description: Build, install on the seed via MCP, and smoke-test a cog with one --once invocation against live sensor data.
arguments:
  - name: id
    description: Cog id to test (defaults to current directory's cog if a `cog.toml` is present)
    required: false
---

# Steps

1. Resolve cog id from `{id}` or by reading `cog.toml` in the current directory.
2. Load the `cog-testing` skill.
3. Spawn the `cog-tester` subagent with the id.
4. The tester runs (in order):
   - `cargo check -p cog-{id}` from `external/cogs/`
   - `cargo build -p cog-{id} --release --target arm-unknown-linux-gnueabihf` (cross-compile)
   - Upload to `gs://cognitum-apps/cogs/arm/cog-{id}-arm` (only if signed in to gcloud; otherwise skip)
   - Over MCP `cognitum-seed`:
     - `seed.cogs.install({id})` — pulls from GCS or local cache
     - `seed.cogs.start({id})`
     - Wait 3 s
     - `seed.cogs.logs({id})` — verify >0 frames emitted
     - `seed.cogs.stop({id})`
5. Report each step's outcome. If any fails, the tester explains why and what to fix.

# Notes

- This delegates all on-device work to the seed via MCP — there's no SSH, no scp, no remote shell.
- For the cross-compile to skip GCS upload, run `gcloud auth revoke --all` first (or use `--no-upload`).
- Real sensor data (not synthetic) requires an ESP32 paired and broadcasting; seed-stream falls back to synthetic in virtual mode.
