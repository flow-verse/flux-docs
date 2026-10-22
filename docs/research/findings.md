# Research findings that shape Flux

Condensed from four internal research reports (FOSS Ableton alternatives; Ableton + frontier tech; DJ performance tech; AI DJ infrastructure). Numbers are as stated in those reports and are unverified here.

## Where FOSS already matches Live 12 Suite
| Live feature | FOSS equivalent |
|---|---|
| Complex Pro warping | Signalsmith Stretch (MIT); Rubber Band (GPL) |
| Stem separation (12.3) | Demucs (htdemucs) |
| Audio-to-MIDI | Basic Pitch (notes + pitch bend) |
| Max for Live | Plugdata (+ELSE, Cyclone), Cardinal |
| Wavetable/Analog, Operator, Sampler | Surge XT, Six Sines, DecentSampler |
| Roar/Amp | Neural Amp Modeler (slimmable) |
| Generative MIDI tools | GenerativeMIDI, Euclid3an Arp, Sparkit, LibreArp |
| Push/Launchpad/APC | DrivenByMoss (Java; manual install) |
| Session view | Ardour Cue window; Zrythm clip launcher |

The gap is integration and UX, not DSP.

## Constraints that change the design
1. **Stem separation is batch.** Production-quality models are not usable on a live stream. Pre-process, load as clips.
2. **Neural synthesis latency.** RAVE needs causal mode live; receptive field ~1 s, jitter up to ~46 ms on Apple Silicon; real-time factor must stay < 1.0 under contention. AFTER adds pitch control but costs more latency/CPU.
3. **Audio thread is sacred.** Inference runs on a normal-priority thread and hands off via SPSC ring; underrun → silence/dry. Reference design: MRT2-Accompany (25 fps generate-ahead, music-position-indexed ring, PDC for fixed latency).
4. **Latency budgets.** Electronic performers tolerate ~30-40 ms; vocalists on IEMs need < 5-7 ms; spatial rendering target < 10 ms.
5. **Edge inference.** Use FP16 on Jetson Orin Nano; INT8 caused a 2.7x regression in one reported case.
6. **Cue-point logic.** Novelty (SSM, CQT 84 bins, 8-bar checkerboard kernel) ∩ metrical (4/8-bar grid) with a salience gate (next 4 bars ≥ 40% of max harmonic energy).
7. **Failover.** Continuous inaudible control tone (LifeSine-style) monitored by PlayAudio12-class hardware.

## Legal / ethics
- **Copyright:** extracting stems from copyrighted recordings still requires clearance for commercial release/broadcast. Prefer models with documented training-data provenance.
- **Biometric privacy (Saudi PDPL):** the reports argue aggregate-only metadata computed in volatile memory falls outside "personal data". Treat as a hypothesis for counsel: the capture/processing step itself may still be regulated, and a DPIA/DPO may apply at venue scale. Keep any facial recognition (e.g. security systems) fully separate from musical decisions. Prefer mmWave radar or non-identifying density estimation over face-based methods.

## Out of scope for now (noted)
Magenta/MusicVAE symbolic generation (Max for Live based), XR/gesture controllers, haptic monitoring (SUBPAC), BCI/EEG, networked remote performance (>40 ms breaks synchronisation).

## Open questions
- Zrythm's implementation language/toolkit differs between sources (C/GTK vs C++/Qt in v2). Verify against the pinned release before choosing host build tooling.
