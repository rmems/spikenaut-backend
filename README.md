<p align="center">
  <img src="docs/logo.png" width="220" alt="Spikenaut">
</p>

<h1 align="center">spikenaut-backend</h1>
<p align="center">Pluggable backend abstraction for spiking neural networks</p>

<p align="center">
  <a href="https://crates.io/crates/spikenaut-backend"><img src="https://img.shields.io/crates/v/spikenaut-backend" alt="crates.io"></a>
  <a href="https://docs.rs/spikenaut-backend"><img src="https://docs.rs/spikenaut-backend/badge.svg" alt="docs.rs"></a>
  <img src="https://img.shields.io/badge/license-GPL--3.0-orange" alt="GPL-3.0">
</p>

---

Trait-based backend layer for SNN applications. Swap between a pure-Rust reference
implementation and a Julia co-process via ZMQ IPC — all behind a unified
`TraderBackend` interface.

## Features

- `TraderBackend` trait — `process_signals`, `initialize`, `save_state`, `reset`, `get_spike_states`
- `RustBackend` — push-pull encoded 8→16 channel reference implementation (no deps)
- `ZmqBrainBackend` *(feature `zmq`)* — reads 88-byte NERO packets from a Julia brain over IPC
- `NeroManifoldSnapshot` — typed view of dopamine / cortisol / acetylcholine / tempo
- `BackendFactory::create(BackendType)` — runtime backend selection

## Installation

```toml
# Cargo.toml

# Pure-Rust only:
spikenaut-backend = "0.1"

# With ZMQ IPC support:
spikenaut-backend = { version = "0.1", features = ["zmq"] }
```

## Quick Start

```rust
use spikenaut_backend::{BackendFactory, BackendType, GpuTelemetry};

let mut backend = BackendFactory::create(BackendType::Rust);
backend.initialize(None).unwrap();

let signals = [0.1, -0.3, 0.5, 0.0, 0.2, -0.1, 0.4, -0.2];
let telem   = GpuTelemetry::default();
let output  = backend.process_signals(&signals, 0.0, &telem).unwrap();
// output[0..16] — lobe readout
```

### ZMQ IPC (Julia brain)

```rust
use spikenaut_backend::{BackendFactory, BackendType};

let mut backend = BackendFactory::create(BackendType::ZmqBrain);
backend.initialize(None).unwrap(); // connects to ipc:///tmp/spikenaut_readout.ipc
let output = backend.process_signals(&signals, 0.0, &telem).unwrap();
// output[16..20] — [dopamine, cortisol, acetylcholine, tempo]
```

## Wire Format (88-byte NERO packet)

```
[0..8]   tick     i64 LE   monotonic tick counter
[8..72]  readout  16×f32   lobe output signals
[72..88] nero     4×f32    [dopamine, cortisol, acetylcholine, tempo]
```

Legacy 72-byte packets (no NERO suffix) are accepted with zero-order hold on the last NERO state.

## Part of the Spikenaut Ecosystem

| Library | Purpose |
|---------|---------|
| [spikenaut-fpga](https://github.com/rmems/spikenaut-fpga) | Q8.8 parameter export + UART readback |
| [spikenaut-encoder](https://github.com/rmems/spikenaut-encoder) | Sensor → spike train conversion |
| [spikenaut-reward](https://github.com/rmems/spikenaut-reward) | Homeostatic reward computation |
| [spikenaut-router](https://github.com/rmems/spikenaut-router) | SNN-based sparse domain routing |
| [neuromod](https://crates.io/crates/neuromod) | Neuromodulator dynamics (published) |

## License

GPL-3.0-or-later
