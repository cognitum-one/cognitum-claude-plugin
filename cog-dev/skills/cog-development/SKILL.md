---
name: cog-development
description: How to author a Cognitum cog. Covers cog.toml schema, cog-sensor-sources, project layout, and the runtime contract.
---

# Cog anatomy

Every cog lives at `external/cogs/src/cogs/<id>/` with three files:

```
<id>/
├── Cargo.toml      # binary name `cog-<id>`, ARM target
├── cog.toml        # manifest: metadata + [config] schema + sensor source default
└── src/main.rs     # entrypoint
```

# `cog.toml`

```toml
[cog]
id = "presence"
name = "Presence Detector"
description = "WiFi CSI-based occupancy detection"
version = "1.0.0"
category = "security"
binary = "cog-presence-arm"
sensor_source_default = "auto"   # auto | seed-stream | esp32-uart | esp32-udp

[[config]]
key = "threshold"
label = "Detection threshold"
type = "number"
default = 2.5
min = 0.5
max = 10.0
unit = "dB"
description = "RSSI variance threshold above which presence is asserted."

[[config]]
key = "interval"
label = "Sampling interval"
type = "integer"
default = 5
min = 1
max = 60
unit = "seconds"

[[allowed_commands]]
name = "calibrate"
description = "Calibrate the noise floor against the current environment."
```

The [config] block drives the cog-store Configure modal **and** `seed.cogs.config_get/set` MCP tools — same schema, two consumers.

# `Cargo.toml` template

```toml
[package]
name = "cog-presence"
version = "1.0.0"
edition = "2021"

[[bin]]
name = "cog-presence-arm"
path = "src/main.rs"

[dependencies]
cog-sensor-sources = { path = "../../../crates/cog-sensor-sources" }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

[profile.release]
opt-level = "s"
lto = true
codegen-units = 1
panic = "abort"
strip = true
```

# `src/main.rs` skeleton

```rust
use cog_sensor_sources::fetch_sensors;
use std::time::Duration;

fn main() {
    let interval = std::env::args().skip_while(|a| a != "--interval")
        .nth(1).and_then(|s| s.parse::<u64>().ok()).unwrap_or(5);
    loop {
        match fetch_sensors() {
            Ok(snap) => {
                let result = process(&snap);
                println!("{}", serde_json::to_string(&result).unwrap_or_default());
            }
            Err(e) => eprintln!("sensor error: {e}"),
        }
        std::thread::sleep(Duration::from_secs(interval));
    }
}

fn process(snap: &serde_json::Value) -> serde_json::Value {
    // ADR-091 — `snap` has the same shape as /api/v1/sensor/stream
    serde_json::json!({"occupied": false, "rssi_variance": 0.0})
}
```

# Constraints

- Output **one JSON object per line on stdout** — the seed's log capture treats this as the cog's data stream.
- Errors go to stderr.
- Don't open arbitrary network sockets — sensor input must come through `cog-sensor-sources`.
- Stay under 5 MB; profile.release settings above achieve this for typical DSP cogs.
