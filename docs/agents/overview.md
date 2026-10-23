# Agents in Flux

Flux lets AI agents (Claude Code, other MCP clients, or your own scripts) compose, arrange, generate and render music. Agents never run inside the audio engine. They talk to Flux over local, structured interfaces, and every edit they make is an ordinary undoable step.

## The three interfaces

| Layer | Repo | What it does |
|---|---|---|
| `flux-engine serve` | flux-host (`flux/engine/`) | JSON-RPC 2.0 over stdio. Creates and edits projects (tracks, plugins, MIDI notes, tempo, markers), renders offline, undo and redo. Objects are addressed by UUID, not index. |
| `flux-ai` | flux-ai-bridge | Job CLI and warm worker (`flux-ai serve`). Generation backends (`generate`), stem separation, transcription and the long-form orchestrator (`longform`). Newline-delimited JSON events, protocol v2. |
| `flux-mcp` | flux-ai-bridge | Model Context Protocol server that exposes a small, curated tool set over the two layers above. |

Why this shape: the one comparative study of agent-driven DAWs (DAWZY, 2025) found agents succeed far more often with a small ID-addressed API, fresh project state before each edit, units converted by the tool rather than the model, and atomic undoable edits. A large OSC surface did worse.

## Safety rules

- **Local only.** The engine listens on stdio; nothing binds a network port. Remote access would also trigger AGPL network-source duties.
- **Undo turns.** An agent wraps its edits in `begin_agent_turn` / `end_agent_turn`, so one undo reverts the whole turn.
- **Dry runs.** Every mutating call accepts `dry_run: true` and reports what would change.
- **Real-time safety.** Edits are applied on the main thread with the engine paused; renders run through the offline renderer. Nothing new runs on the audio thread.
- **Disclosure by default.** Generated audio always carries AI-generated tags and a provenance record (see [distribution.md](distribution.md)).

## Long-form music ("4 hours of deep house")

No model generates hours of coherent music on its own. Flux's orchestrator plans the set first (energy arc, key path, tempo path, sections), generates most of the music as notes on an exact beat grid, renders it, optionally layers neural audio per section, joins sections with beat-aligned transitions, checks for repetition, and masters the whole mix to −14 LUFS / −1 dBTP. See `flux-ai-bridge/docs/longform.md`.

## Further reading

- [backends.md](backends.md): generation models, licences and measured speed on Apple Silicon.
- [distribution.md](distribution.md): disclosure, provenance and platform rules.
- `flux-host/flux/engine/API.md`: engine method reference.
- `flux-ai-bridge/docs/protocol.md`: job protocol v2.
- `flux-ai-bridge/docs/mcp.md`: MCP server setup (`claude mcp add flux -- flux-mcp …`), tool reference and eval results.
- `flux-ai-bridge/docs/longform.md`: long-form orchestrator usage and measured runs.
