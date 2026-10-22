# Architecture overview

Flux is an orchestrator over existing FOSS audio tech. Polyglot by design:

| Layer | Language | Why |
|---|---|---|
| Host (flux-host) | C / C++ | Fork of Zrythm; deterministic real-time audio, no GC |
| AI bridge (flux-ai-bridge) | Rust (nih-plug, PyO3) + Python | Safe CLAP plugin; PyTorch models stay in Python |
| Stage (flux-stage) | Rust | Sync, DMX, spatial, failover |
| Edge (flux-edge) | Python (TensorRT) | Jetson inference |
| Installer | Bash + YAML | OS config, Flatpak/Docker |

Rule: no garbage collection or locks on the audio thread. Do not port the Zrythm core to another language.
