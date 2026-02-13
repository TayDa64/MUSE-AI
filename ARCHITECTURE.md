# MUSE-ai — System Architecture

> **Music Understanding and Synthesis Engine**  
> **Date**: February 10, 2026

---

## Overview

MUSE-ai is a three-component AI music production system that merges:

1. **copilot-Liku-cli** — Electron-based agent CLI for natural language interaction
2. **MUSE** (formerly multimodal-ai-music-gen) — Python intelligence + JUCE audio frontend
3. **MPC Beats** — Akai Professional DAW as the final rendering surface

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              USER                                       │
│                                                                         │
│     Types: "dark neo-soul with gospel influences at 75 BPM"            │
└────────────────────────────┬────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  COPILOT-LIKU-CLI (Electron)                                            │
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌────────────┐                │
│  │  Supervisor   │───→│   Builder    │───→│  Verifier  │                │
│  │  Agent        │    │   Agent      │    │  Agent     │                │
│  │              │    │              │    │            │                │
│  │ Decomposes   │    │ Executes via │    │ Runs       │                │
│  │ NL intent    │    │ Python calls │    │ critics    │                │
│  └──────────────┘    └──────┬───────┘    └─────┬──────┘                │
│                             │                  │                        │
│  ┌──────────────┐           │                  │                        │
│  │  Researcher  │           │                  │                        │
│  │  Agent       │           │                  │                        │
│  │              │           │                  │                        │
│  │ Genre/ref    │           │                  │                        │
│  │ analysis     │           │                  │                        │
│  └──────────────┘           │                  │                        │
│                             │                  │                        │
│  Overlay ─── Chat ─── Tray │                  │                        │
└─────────────────────────────┼──────────────────┼────────────────────────┘
                              │ JSON-RPC         │ JSON-RPC
                              ▼                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  MUSE PYTHON ENGINE (persistent process)                                │
│                                                                         │
│  ┌─────────── intelligence/ ──────────────────────────────────┐        │
│  │                                                             │        │
│  │  ┌──────────────┐  ┌──────────┐  ┌───────────┐            │        │
│  │  │ HarmonicBrain│  │ GenreDNA │  │ Critics   │            │        │
│  │  │              │  │          │  │           │            │        │
│  │  │ • Voice lead │  │ • 10-dim │  │ • VLC     │            │        │
│  │  │ • Voicing    │  │ • Fusion │  │ • RCS     │            │        │
│  │  │ • Tension    │  │ • Interp │  │ • BKAS    │            │        │
│  │  │ • Roman nums │  │          │  │ • VDQ     │            │        │
│  │  └──────────────┘  └──────────┘  │ • ADC     │            │        │
│  │                                   └───────────┘            │        │
│  │  ┌──────────────┐  ┌──────────┐  ┌───────────┐            │        │
│  │  │ MPC Corpus   │  │Embeddings│  │Preferences│            │        │
│  │  │              │  │          │  │           │            │        │
│  │  │ • 47 progs   │  │ • ONNX   │  │ • Vectors │            │        │
│  │  │ • 80 arps    │  │ • Search │  │ • Modes   │            │        │
│  │  │ • Profiles   │  │ • Index  │  │ • History │            │        │
│  │  └──────────────┘  └──────────┘  └───────────┘            │        │
│  └─────────────────────────────────────────────────────────────┘        │
│                                                                         │
│  ┌─────────── agents/ ────────────────────────────────────────┐        │
│  │                                                             │        │
│  │  OfflineConductor                                          │        │
│  │    │                                                       │        │
│  │    ├─→ DrummerAgent ───→ PerformanceResult (notes[])       │        │
│  │    ├─→ BassistAgent ───→ PerformanceResult                 │        │
│  │    ├─→ KeyboardistAgent → PerformanceResult                │        │
│  │    ├─→ KrarAgent ──────→ PerformanceResult                 │        │
│  │    ├─→ KeberoAgent ────→ PerformanceResult                 │        │
│  │    ├─→ MasenqoAgent ──→ PerformanceResult                 │        │
│  │    ├─→ WashintAgent ──→ PerformanceResult                 │        │
│  │    └─→ BegenaAgent ───→ PerformanceResult                 │        │
│  │                                                             │        │
│  │  All performers receive enriched PerformanceContext         │        │
│  │  containing MUSE intelligence fields                       │        │
│  └─────────────────────────────────────────────────────────────┘        │
│                                                                         │
│  ┌─────────── pipeline ───────────────────────────────────────┐        │
│  │  PromptParser → Arranger → MidiGenerator → AudioRenderer   │        │
│  │  → MPCExporter → session_graph.SessionGraph                │        │
│  └─────────────────────────────────────────────────────────────┘        │
│                                                                         │
│  ┌─────────── server/ ────────────────────────────────────────┐        │
│  │  osc_server.py (→ JUCE)    jsonrpc_server.py (→ CLI)      │        │
│  └─────────────────────────────────────────────────────────────┘        │
└───────────┬──────────────────────────────────┬──────────────────────────┘
            │ OSC (UDP 9000/9001)               │ Virtual MIDI (loopMIDI)
            ▼                                   ▼
┌───────────────────────┐           ┌────────────────────────────────────┐
│  JUCE FRONTEND        │           │  MPC BEATS (Akai DAW)              │
│  (passive viewport)   │           │                                    │
│                       │           │  Receives MIDI on virtual port     │
│  • Piano roll display │           │  • TubeSynth, Bassline, Electric   │
│  • Waveform view      │           │  • 30+ AIR effects                 │
│  • Spectrum analyzer  │           │  • Professional mixing             │
│  • Transport sync     │           │  • Export to WAV/AIFF              │
│  • 16-voice synth     │           │                                    │
│    (preview only)     │           │  The FINAL audio surface           │
└───────────────────────┘           └────────────────────────────────────┘
```

---

## Communication Protocols

### copilot-Liku-cli ↔ MUSE Python

**Protocol**: JSON-RPC 2.0 over stdio (child_process.spawn) or local HTTP

```
CLI spawns Python as persistent subprocess:
  child_process.spawn('python', ['-m', 'multimodal_gen.server.jsonrpc_server'])

Request:
  { "jsonrpc": "2.0", "method": "generate", "params": {...}, "id": 1 }

Response (streaming):
  { "jsonrpc": "2.0", "result": { "progress": 0.5, "stage": "voicing" }, "id": 1 }
  { "jsonrpc": "2.0", "result": { "tracks": [...], "critics": {...} }, "id": 1 }
```

### MUSE Python ↔ JUCE Frontend

**Protocol**: OSC over UDP (port 9000: Python→JUCE, port 9001: JUCE→Python)

Already implemented. Used for passive display updates:
- `/progress` — generation progress
- `/complete` — generation done with track data
- Piano roll note data, transport state, FX chain updates

### MUSE Python ↔ MPC Beats

**Protocol**: Virtual MIDI via loopMIDI + rtmidi

```
Python → loopMIDI virtual port "MUSE AI Controller" → MPC Beats receives on MIDI input
```

Not yet implemented. Sprint 3 deliverable.

---

## Agent Reconciliation: How MUSE Intelligence Informs Performers

```
MUSE Intelligence (X-axis: cross-cutting concerns)
─────────────────────────────────────────────────────────
│             │ Harmony    │ Rhythm     │ Sound      │ Mix
│             │ (Brain)    │ (DNA)      │ (Oracle)   │ (Engineer)
─────────────────────────────────────────────────────────
│ Drummer     │ tension→   │ pattern,   │ kit        │ compression,
│             │ fill timing│ swing,     │ selection  │ room verb
│             │            │ density    │            │
│ Bassist     │ chord      │ groove     │ bass synth │ sidechain,
│             │ tones,     │ lock,      │ preset     │ sub freq
│             │ approaches │ swing      │            │
│ Keyboardist │ voicings,  │ chord      │ keys       │ EQ, stereo
│             │ voice lead │ rhythm     │ preset     │ width
│             │            │            │            │
│ Lead/Krar   │ scale,     │ phrasing   │ timbre     │ delay,
│             │ tension    │ density    │            │ presence
─────────────────────────────────────────────────────────

MUSE agents don't generate notes — they CONFIGURE performers.
Performers don't make harmonic decisions — they EXECUTE with personality.
```

### Data Flow Per Generation

```
1. User prompt arrives at copilot-Liku-cli
2. SupervisorAgent decomposes: genre, mood, tempo, key, instruments
3. ResearcherAgent queries GenreDNA + reference analysis
4. BuilderAgent sends generation request to Python:
   a. HarmonicBrain generates progression + voicings
   b. GenreDNA produces 10-dim vector → groove templates, rhythm params
   c. OfflineConductor builds PerformanceContext (enriched)
   d. Performers execute sequentially (drums → bass → keys → lead)
   e. Critics score output (VLC, RCS, BKAS, VDQ, ADC)
   f. If critics fail: regenerate failing component (max 3 attempts)
5. VerifierAgent validates quality report
6. Output: MIDI files + SessionGraph manifest + critic report
7. Optional: Virtual MIDI playback into MPC Beats
```

---

## File Organization

```
MUSE-ai/
├── ROADMAP.md                          # ← Implementation plan (this sprint)
├── ARCHITECTURE.md                     # ← This document
│
├── copilot-Liku-cli/                   # Layer 1: Intent
│   ├── src/
│   │   ├── main/
│   │   │   ├── agents/
│   │   │   │   ├── orchestrator.js     # Routes to supervisor/builder/verifier
│   │   │   │   ├── supervisor.js       # NL → musical specs
│   │   │   │   ├── builder.js          # Calls Python intelligence
│   │   │   │   ├── verifier.js         # Quality gate
│   │   │   │   ├── researcher.js       # Genre research
│   │   │   │   └── state-manager.js    # Session persistence
│   │   │   └── index.js               # Electron main process
│   │   ├── cli/
│   │   │   └── liku.js                # CLI entry point
│   │   └── renderer/                  # Overlay + Chat UI
│   └── package.json
│
├── MUSE/                               # Layer 2+3: Intelligence + Performance
│   ├── multimodal_gen/
│   │   ├── intelligence/               # ← NEW: MUSE brain modules
│   │   │   ├── harmonic_brain.py       # Voice leading, voicing, tension
│   │   │   ├── genre_dna.py            # 10-dim vectors, fusion
│   │   │   ├── mpc_corpus.py           # .progression + .mid parsing
│   │   │   ├── critics.py              # VLC, RCS, BKAS, VDQ, ADC
│   │   │   ├── embeddings.py           # ONNX vector search
│   │   │   ├── midi_bridge.py          # Virtual MIDI to MPC Beats
│   │   │   └── preferences.py          # User preference model
│   │   ├── agents/                     # Existing performer system
│   │   │   ├── performers/             # 8 agents (drums, bass, keys, ethiopian)
│   │   │   ├── conductor_offline.py    # Orchestration engine
│   │   │   └── context.py             # PerformanceContext (to be enriched)
│   │   ├── server/
│   │   │   ├── osc_server.py           # → JUCE communication
│   │   │   └── jsonrpc_server.py       # ← NEW: → CLI communication
│   │   └── ...                         # Existing pipeline modules
│   ├── juce/                           # C++ audio frontend (frozen)
│   ├── docs/
│   │   ├── muse-specs/                 # MUSE design bible (91 tasks, 15 contracts)
│   │   └── AGENT_ARCHITECTURE_PLAN.md  # Existing agent design doc
│   └── main.py                         # Python entry point
│
└── expansions/                         # MPC expansion packs
```

---

## Key Design Decisions (Inherited + New)

| ID | Decision | Resolution | Source |
|---|---|---|---|
| D-001 | CLI vs Extension API | CLI JSON-RPC (copilot-Liku-cli is Electron CLI) | MUSE spec |
| D-002 | Agent count | 4 agents in CLI (Supervisor, Builder, Verifier, Researcher) + 8 Python performers | Adapted |
| D-003 | Embedding model | Local ONNX (all-MiniLM-L6-v2, 384-dim) | MUSE spec |
| D-004 | Virtual MIDI | loopMIDI + rtmidi | MUSE spec |
| D-005 | Quality thresholds | Strict (VLC < 3.0, RCS 0.7-0.95, BKAS > 70%) | MUSE spec |
| D-009 | Project name | MUSE (Music Understanding and Synthesis Engine) | MUSE spec |
| D-010 | State dependency | SessionGraph NEVER depends on conversation memory | MUSE spec |
| NEW-1 | JUCE strategy | Frozen as passive viewport for 8 weeks | Synthesis report |
| NEW-2 | Primary interface | copilot-Liku-cli Electron chat | This architecture |
| NEW-3 | Intelligence language | Python (not TypeScript, despite MUSE specs) | Synthesis report |
| NEW-4 | Offline-first | All musical intelligence works without cloud LLM | All reviewers |

---

*This architecture document is the technical companion to [ROADMAP.md](ROADMAP.md). It describes HOW the components connect; the roadmap describes WHEN things get built.*
