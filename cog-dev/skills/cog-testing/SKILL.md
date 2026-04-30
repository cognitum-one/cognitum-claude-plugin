---
name: cog-testing
description: Test loop for a cog: cargo check, ARM cross-compile, on-seed install/start/smoke via MCP.
---

# Pre-flight

```bash
cargo check -p cog-{id}
```

Resolve obvious type / import errors before the slow ARM build.

# ARM cross-compile

```bash
# Inside external/cogs/
cargo build -p cog-{id} --release --target arm-unknown-linux-gnueabihf
```

The artifact lands at `target/arm-unknown-linux-gnueabihf/release/cog-{id}`. Strip + check size:

```bash
ls -la target/arm-unknown-linux-gnueabihf/release/cog-{id}
# Expect <5 MB
```

# Install + smoke (over MCP)

```jsonc
// All over the cognitum-seed MCP connection:
seed.cogs.install({"id": "{id}"})    // Pulls from local cache or GCS
seed.cogs.start({"id": "{id}"})
// wait 3 s
seed.cogs.logs({"id": "{id}", "lines": 20})
// expect >0 JSON lines emitted
seed.cogs.stop({"id": "{id}"})
```

# What "passing" means

- `cargo check` clean (no errors)
- ARM artifact exists and is <5 MB
- `seed.cogs.install` returns `{"status":"installed"}` or `{"status":"already_installed"}`
- `seed.cogs.start` returns a PID
- `seed.cogs.logs` returns ≥1 JSON object after 3 s

# Failure modes

- "cog not installed" after install → check binary name matches `cog-{id}-arm` exactly
- 0 lines emitted → cog crashed silently; inspect `seed.cogs.logs` for stderr lines
- Cog runs but no sensor data → check `seed.sensor.snapshot` returns frames; if not, ESP32 may be unpaired
