# Generation backends

Flux downloads model weights on demand; no weights ship in any repository or installer. Each model's licence is shown and accepted once (`flux-ai generate --accept-licence <id>`), and the acceptance is recorded in `~/.cache/flux-ai/licences.json`. Every output gets a `.provenance.json` sidecar.

## Licence policy

| Backend | Code | Weights | Default | Notes |
|---|---|---|---|---|
| ACE-Step 1.5 (turbo, xl-turbo) | MIT | MIT | **On** | Full songs up to 10 min, vocals, BPM/key/lyrics conditioning, repaint, continue. Training data not documented. |
| Magenta RealTime 2 (base, small) | Apache-2.0 | CC-BY-4.0 | **On** | Streaming instrumental model with MIDI/text/audio control. Needs MLX 0.31.2 on macOS. |
| Anticipatory Music Transformer | Apache-2.0 | check model card | **On** (symbolic) | MIDI generate, continue, infill. |
| Stable Audio 3 (small-music, medium, small-sfx) | MIT (MLX runtime) | Stability AI Community Licence (+ Gemma terms for its text encoder) | Opt-in | Trained on licensed (AudioSparx) and Creative Commons (Freesound) audio. Free under $1M annual revenue; register for commercial use; show "Powered by Stability AI". Runs through Stability's MLX runtime on Apple Silicon. |
| Non-commercial models (MusicGen, YuE2, …) | varies | CC BY-NC | Off | No adapters shipped. |
| Cloud APIs (Suno, Lyria, …) | — | — | Off | Not supported: Flux stays local and free. |

The weight licence governs your use of the output, not Flux's code licence. This is not legal advice.

## Measured on a Mac Studio (M1 Ultra, 64 GB)

120 s of 122 BPM A-minor instrumental, 5 seeds per tier, warm model. Full method and caveats: `flux-ai-bridge/docs/benchmarks/mac-studio-m1-ultra.md`.

| Backend / tier | Compute per audio minute | Real-time factor | BPM within ±1 | Key correct (incl. relative major) |
|---|---|---|---|---|
| ACE-Step turbo | 34.7 s | 0.58 | 5/5 | 3/5 |
| ACE-Step xl-turbo | 42.6 s | 0.71 | 5/5 | 4/5 |
| Magenta RT 2 base (with note guide) | 83.3 s | 1.39 | 2/5 | 5/5 |
| Magenta RT 2 small (with note guide) | 51.4 s | 0.86 | 1/5 | 5/5 |
| Stable Audio 3 small-music (MLX) | 1.7 s | 0.029 | 3/5 | 2/5 |
| Stable Audio 3 medium (MLX) | 4.4 s | 0.073 | 5/5 | 0/5 |

Takeaways:

- ACE-Step is the default for neural layers: faster than real time and holds tempo. Key adherence is not yet at the 80% target.
- Magenta RT 2 has no tempo input; Flux steers it with a MIDI "note guide", which fixes the key but not the tempo. On the M1 Ultra it is not real time in Python, so live streaming (roadmap phase 8, optional) needs an M2 Max or newer, or a faster runtime.
- Stable Audio 3 is 7-17x faster than ACE-Step on this Mac and Medium holds tempo, but it ignores a key given in the prompt: use it for key-neutral layers (drums, textures, effects) and drafts.
- For long-form work, the beat grid comes from the symbolic layer; neural audio is layered on top and checked for tempo and key.
