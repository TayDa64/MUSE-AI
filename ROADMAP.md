# MUSE-ai — Integration Roadmap

> **Music Understanding and Synthesis Engine**  
> **Created**: February 10, 2026  
> **Status**: ACTIVE — Grounded against existing codebase  
> **Source Specs**: [docs/muse-specs/](MUSE/docs/muse-specs/) (91 tasks, 15 contracts, 14 decisions)  
> **Synthesis Report**: [MUSE-INTEGRATION-SYNTHESIS.md](MUSE/docs/muse-specs/MUSE-INTEGRATION-SYNTHESIS.md)

---

## Latest Status Update (Feb 22, 2026)

### Completed Integration Milestones
- ✅ Copilot model orchestration phases 0–5 implemented in the Electron ↔ Python flow.
- ✅ Reference-grounding bridge wired (`analyze_reference` JSON-RPC + producer context injection).
- ✅ Role-specialized model policy active (director/producer/verifier).
- ✅ Structured handoff validation/fallback added before generation.
- ✅ Preflight verifier + post-run critics/output analysis integrated.
- ✅ JUCE-compatible phase progress states added (`step`, `percent`, `message`, `timestamp`).

### Reliability Fixes Applied
- ✅ `PythonBridge` default `cwd` validated for this workspace layout.
- ✅ Worker cancellation/result consistency fixed (terminal state always has result payload).
- ✅ `get_status` now falls back to worker result when cache misses.
- ✅ `generate_sync` calls now use explicit long RPC timeout in producer path.
- ✅ Runtime Unicode-arrow logging crash (`charmap` on Windows consoles) patched in active path.

### Current Runtime Behavior
- Song generation is completing and producing MIDI/WAV outputs.
- If critics fail, producer returns a graceful failure state by default (`COMPLETED_WITH_CRITIC_FAIL`).
- Optional bypass for smoke/throughput testing is available:
    - `/produce --accept-generation <prompt>`
    - `/produce --allow-critic-fail <prompt>`

### Immediate Next (pre-commit)
- Run one strict `/produce` and one bypass `/produce --accept-generation` smoke check.
- Stage and commit roadmap + producer/chat reliability updates together.

---

## Project Structure

```
MUSE-ai/
├── MUSE/                           # Python + JUCE music generation engine
│   ├── multimodal_gen/             # Python backend (intelligence + generation)
│   │   ├── agents/                 # 8 performer agents + conductor
│   │   ├── intelligence/           # ← NEW: MUSE brain modules
│   │   └── ...                     # Existing pipeline modules
│   ├── juce/                       # C++ real-time audio frontend
│   └── docs/muse-specs/            # MUSE design bible (copied from MPC Beats)
│
├── copilot-Liku-cli/               # Electron agent CLI (Layer 1: Intent)
│   └── src/main/agents/            # Orchestrator, Supervisor, Builder, Verifier
│
└── expansions/                     # MPC Beat expansion packs
```

---

## 3-Layer Architecture: What Maps Where

```
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 1: INTENT                    copilot-Liku-cli/               │
│                                                                      │
│  AgentOrchestrator → SupervisorAgent → BuilderAgent → VerifierAgent │
│  ─ NL understanding, routing, session management                     │
│  ─ Supervisor = intent decomposition                                 │
│  ─ Builder = calls Python intelligence                               │
│  ─ Verifier = runs critics / quality gate                            │
│  ─ Researcher = genre research, reference analysis                   │
│                                                                      │
│  MUSE equivalent: customAgents with infer:true routing               │
│  Status: SCAFFOLD EXISTS — agents have structure, need music wiring  │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ IPC / JSON-RPC / OSC
                           ▼
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 2: DOMAIN INTELLIGENCE       MUSE/multimodal_gen/            │
│                                                                      │
│  ┌────────────────┐  ┌──────────────┐  ┌───────────┐  ┌──────────┐ │
│  │ HarmonicBrain  │  │  GenreDNA    │  │ Embeddings│  │ Critics  │ │
│  │ (voice leading │  │  (10-dim     │  │ (ONNX     │  │ (5-axis  │ │
│  │  tension func  │  │   vectors    │  │  search)  │  │  scoring)│ │
│  │  voicing)      │  │   fusion)    │  │           │  │          │ │
│  └────────────────┘  └──────────────┘  └───────────┘  └──────────┘ │
│                                                                      │
│  Status: PARTIALLY EXISTS — see mapping table below                  │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ PerformanceContext (enriched)
                           ▼
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 3: PERFORMANCE               MUSE/multimodal_gen/agents/     │
│                                                                      │
│  OfflineConductor → DrummerAgent, BassistAgent, KeyboardistAgent,   │
│                     KrarAgent, KeberoAgent, MasenqoAgent,           │
│                     WashintAgent, BegenaAgent                       │
│                                                                      │
│  Status: IMPLEMENTED — 8 performers, conductor, registry working     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## What Already Exists vs. What MUSE Adds

### Existing Code → MUSE Phase Mapping

| MUSE Concept | MUSE Phase | Existing Module | Status | Gap |
|---|---|---|---|---|
| **Chord progressions** | P1 (Harmonic Brain) | `midi_generator._create_chord_track()` | ⚠️ Lookup tables | No voice leading, no tension function, no roman numeral analysis |
| **Genre intelligence** | P1 (GenreDNA) | `genre_intelligence.py` (716 lines) | ✅ Partial | Has genre templates with tempo/key/elements. Missing: 10-dim vectors, interpolation, fusion |
| **Quality validation** | P8 (Critics) | `quality_validator.py` (1581 lines) | ✅ Solid foundation | Has 7 metric categories, genre profiles, thresholds. Needs: VLC, BKAS, inter-agent metrics |
| **Tension arcs** | P1 (Tension) | `tension_arc.py` (699 lines) | ✅ Good | Has arc shapes, tension curves, config. Needs: composite function with interaction terms |
| **Session state** | P0 (SongState) | `session_graph.py` (1351 lines) | ✅ Superior | Richer than MUSE's SongState. Needs: genre_dna, progression, compression methods (L0-L3) |
| **Performance context** | P0 | `agents/context.py` | ✅ Working | Needs: voicing_constraints, groove_template, genre_dna, orchestration_map |
| **Humanization** | P3 (Rhythm) | `humanize_physics.py`, `drum_humanizer.py`, `microtiming.py` | ✅ Advanced | Physics-based humanization already exists. Needs: genre-specific timing tables |
| **Bass generation** | P3 | `agents/performers/bassist.py` | ⚠️ Template-based | Root transposition only. Needs: chromatic approaches, passing tones, harmonic awareness |
| **Drum generation** | P3 | `agents/performers/drummer.py` | ⚠️ Pattern-based | Pattern selection. Needs: genre-appropriate subdivisions, syncopation from GenreDNA |
| **Keyboard voicing** | P1 (Voicing) | `agents/performers/keyboardist.py` | ❌ No voice leading | Generates chords without reference to previous voicing. #1 priority fix |
| **Groove templates** | P3 | `groove_templates.py` | ✅ Exists | Expand with MPC corpus analysis (80 arp patterns → swing/density profiles) |
| **Style policy** | P6 | `style_policy.py` | ✅ Good | PolicyContext with timing, voicing, dynamics, arrangement, mix, instrument policies |
| **Arrangement** | P6 | `arranger.py` | ⚠️ Basic | Needs: instrument activation per section, density curves, genre-specific forms |
| **Take system** | P8 | `take_generator.py` | ✅ Exists | Variation generation with seeds |
| **Reference analysis** | P2 | `reference_analyzer.py`, `file_analysis.py` | ✅ Exists | BPM/key extraction, audio analysis |
| **Mix engine** | P4 | `mix_chain.py`, `mix_engine.py`, `auto_gain_staging.py` | ✅ Extensive | Multiple processors, EQ, dynamics, spatial audio |
| **Preset system** | P4 | `preset_system.py`, `instrument_index.py`, `instrument_intelligence.py` | ✅ Working | Instrument resolution + intelligence layer |
| **MPC export** | P5 | `mpc_exporter.py` | ✅ Complete | Working MPC project export |
| **Ethiopian music** | — | 5 agents + `ethio_melody.py` | ✅ Unique | Qenet theory, krar/kebero/masenqo/washint/begena |
| **OSC bridge** | P5 | `server/osc_server.py` | ✅ Working | Python ↔ JUCE communication |
| **Virtual MIDI** | P5 | Not implemented | ❌ Missing | Need loopMIDI + rtmidi for MPC Beats integration |
| **Embeddings/search** | P2 | Not implemented | ❌ Missing | Need ONNX runtime + vector index for semantic search |
| **MPC data parsing** | P0 | Not implemented | ❌ Missing | 47 .progression files + 80 arp patterns unparsed |
| **Controller intelligence** | P7 | Not implemented | ❌ Deferred | MPC-specific, not critical path |
| **Personalization** | P9 | Not implemented | ❌ Future | Preference vectors, producer modes |
| **Live performance** | P10 | Not implemented | ❌ Future | Real-time MIDI input/harmonization |

### copilot-Liku-cli → MUSE Agent Mapping

| MUSE Agent | copilot-Liku-cli Agent | Mapping |
|---|---|---|
| **Composer (Orchestrator)** | `AgentOrchestrator` | Routes to specialists, manages session |
| **Intent Decomposition** | `SupervisorAgent` | Interprets NL prompts → musical specs |
| **Generation Execution** | `BuilderAgent` | Calls Python intelligence → produces output |
| **Quality Gate** | `VerifierAgent` | Runs critics, validates output quality |
| **Genre Research** | `ResearcherAgent` | Gathers genre info, reference analysis |
| Harmony Agent | — | → Python `HarmonicBrain` (new module) |
| Rhythm Agent | — | → Python agents + humanization (exists) |
| Sound Oracle | — | → Python `instrument_intelligence.py` (exists) |
| Mix Engineer | — | → Python `mix_engine.py` (exists) |
| Controller Agent | — | Deferred (MPC-specific) |

---

## Priority Ordering (Sprint-Based)

### Sprint 1 — Harmonic Foundation (Weeks 1-2)

**Goal**: Chord progressions that sound professional, not student-level.

| # | Task | MUSE Ref | Target File | Complexity |
|---|------|----------|-------------|------------|
| 1.1 | **Parse MPC .progression files** | P0-T4 | `multimodal_gen/intelligence/mpc_corpus.py` | S |
| 1.2 | **Parse MPC .mid arp patterns** | P0-T3 | `multimodal_gen/intelligence/mpc_corpus.py` | M |
| 1.3 | **Build GenreDNA vector table** from corpus | P1-T1/C8 | `multimodal_gen/intelligence/genre_dna.py` | M |
| 1.4 | **Voice leading engine** | P1-T4 | `multimodal_gen/intelligence/harmonic_brain.py` | M |
| 1.5 | **Chord-to-MIDI voicing engine** | P1-T6 | `multimodal_gen/intelligence/harmonic_brain.py` | L |
| 1.6 | **Chord progression generator** (corpus-based) | P1-T7 | `multimodal_gen/intelligence/harmonic_brain.py` | M |
| 1.7 | **Inject into KeyboardistAgent** | — | `multimodal_gen/agents/performers/keyboardist.py` | S |

**Acceptance**: "Neo Soul in Eb minor, 8 chords" → voice-led progression with common-tone retention. VLC < 3.0 avg.

**Data source**: `c:\dev\MPC Beats\Progressions\` (47 files) + `c:\dev\MPC Beats\Arp Patterns\` (80+ files)

### Sprint 2 — Generation Quality (Weeks 3-4)

**Goal**: Bass, drums, and arrangement that respond to harmony and energy.

| # | Task | MUSE Ref | Target File | Complexity |
|---|------|----------|-------------|------------|
| 2.1 | **Harmonic bass line generator** | P3 | `multimodal_gen/agents/performers/bassist.py` | M |
| 2.2 | **Genre-specific drum patterns** | P3 | `multimodal_gen/agents/performers/drummer.py` | M |
| 2.3 | **Genre-specific timing offset tables** | P3-T6 | `multimodal_gen/groove_templates.py` | S |
| 2.4 | **Arrangement: instrument activation per section** | P6 | `multimodal_gen/agents/conductor_offline.py` | M |
| 2.5 | **Enrich PerformanceContext** with MUSE fields | P0 | `multimodal_gen/agents/context.py` | S |
| 2.6 | **Extend quality_validator** with VLC + BKAS + ADC | P8-T2 | `multimodal_gen/quality_validator.py` | M |
| 2.7 | **Targeted regeneration** on critic failure | P8-T3 | `multimodal_gen/agents/conductor_offline.py` | M |

**Acceptance**: 5-genre test (neo-soul, trap, house, lo-fi, Ethiopian). Multi-track MIDI passing 3+ critics.

### Sprint 3 — Integration Bridge (Weeks 5-6)

**Goal**: copilot-Liku-cli drives the Python engine. End-to-end via chat.

| # | Task | MUSE Ref | Target File | Complexity |
|---|------|----------|-------------|------------|
| 3.1 | **Python JSON-RPC server** | P0-T8 | `MUSE/multimodal_gen/server/jsonrpc_server.py` | M |
| 3.2 | **Wire BuilderAgent → Python** | P6-T2 | `copilot-Liku-cli/src/main/agents/builder.js` | M |
| 3.3 | **Wire VerifierAgent → Critics** | P8-T3 | `copilot-Liku-cli/src/main/agents/verifier.js` | S |
| 3.4 | **Wire ResearcherAgent → GenreDNA** | P1 | `copilot-Liku-cli/src/main/agents/researcher.js` | S |
| 3.5 | **Session state management** | P0-T6 | `copilot-Liku-cli/src/main/agents/state-manager.js` | M |
| 3.6 | **Virtual MIDI to MPC Beats** | P5-T1,T2 | `MUSE/multimodal_gen/intelligence/midi_bridge.py` | L |

**Acceptance**: User types prompt in Electron chat → Python generates → output plays in MPC Beats via virtual MIDI.

### Sprint 4 — Intelligence Depth (Weeks 7-8)

**Goal**: Embedding search, personalization hooks, JUCE as passive display.

| # | Task | MUSE Ref | Target File | Complexity |
|---|------|----------|-------------|------------|
| 4.1 | **ONNX embedding index** | P2-T3,T4 | `MUSE/multimodal_gen/intelligence/embeddings.py` | L |
| 4.2 | **Semantic preset search** | P2-T6 | `MUSE/multimodal_gen/intelligence/embeddings.py` | M |
| 4.3 | **SessionGraph L0-L3 compression** | P0/C1 | `MUSE/multimodal_gen/session_graph.py` | M |
| 4.4 | **Genre fusion engine** | P8-T5 | `MUSE/multimodal_gen/intelligence/genre_dna.py` | M |
| 4.5 | **OSC notifications to JUCE** | P5 | `MUSE/multimodal_gen/server/osc_server.py` | S |
| 4.6 | **Preference tracking hooks** | P9-T1 | `MUSE/multimodal_gen/intelligence/preferences.py` | M |

**Acceptance**: "warm Sunday morning vibe" → finds matching presets via embeddings. Genre fusion produces coherent hybrids.

---

## New Module: `multimodal_gen/intelligence/`

This is where MUSE brain modules live. All are pure Python, no LLM dependency, offline-first.

```
multimodal_gen/intelligence/
├── __init__.py
├── harmonic_brain.py       # Voice leading, voicing engine, tension function
│                           # Source: MUSE P1 (8 tasks)
│                           # Contracts: C2 (EnrichedProgression)
│
├── genre_dna.py            # 10-dimensional genre vectors, fusion, interpolation
│                           # Source: MUSE P1/C8 + Synthesis Report §9
│                           # Enhances: existing genre_intelligence.py
│
├── mpc_corpus.py           # Parser for .progression + .mid arp files
│                           # Source: MUSE P0-T3, P0-T4
│                           # Input: c:\dev\MPC Beats\Progressions\ + Arp Patterns\
│                           # Output: EnrichedProgression[], ParsedArpPattern[]
│
├── critics.py              # Multi-axis quality gate (VLC, RCS, BKAS, VDQ, ADC)
│                           # Source: MUSE P8-T2, P8-T3
│                           # Enhances: existing quality_validator.py
│
├── embeddings.py           # ONNX all-MiniLM-L6-v2, Float32Array vector index
│                           # Source: MUSE P2 (7 tasks)
│                           # Contracts: C7 (EmbeddingIndex)
│
├── midi_bridge.py          # Virtual MIDI via rtmidi + loopMIDI detection
│                           # Source: MUSE P5 (8 tasks)
│                           # Contracts: C11 (ControllerMapping)
│
└── preferences.py          # User preference vectors, producer modes
                            # Source: MUSE P9 (8 tasks)
                            # Contracts: C14 (UserPreferences)
```

---

## PerformanceContext Extensions

The `PerformanceContext` dataclass (agents/context.py) is the bridge between MUSE intelligence and existing performers. Add these fields:

```python
# MUSE Intelligence Fields (Sprint 1-2)
voicing_constraints: Optional[VoicingConstraints] = None   # From HarmonicBrain
groove_template: Optional[GrooveTemplate] = None           # From GenreDNA  
genre_dna: Optional[GenreDNAVector] = None                 # 10-dim vector
tension_curve: List[float] = field(default_factory=list)   # Composite function
orchestration_map: Dict[str, bool] = field(default_factory=dict)  # agent → active
previous_voicing: Optional[List[int]] = None               # For voice leading continuity
```

---

## SessionGraph Extensions

Add to `session_graph.py` without breaking existing interface:

```python
# MUSE Fields
genre_dna: Optional[Dict[str, float]] = None      # 10-dim vector as dict
progression: List[Dict] = field(default_factory=list)  # Explicit chord sequence
preferences: Optional[Dict] = None                # C14 UserPreferences
compression_version: str = "1.0.0"

# Compression Methods (MUSE L0-L3)
def to_l0(self) -> str:    # ~100 tokens: "8 tracks, Eb minor, 130 BPM, neo-soul"
def to_l1(self) -> dict:   # ~500 tokens: structural skeleton
def to_l2(self) -> dict:   # ~3000 tokens: full detail
def to_l3(self) -> dict:   # Variable: full history with deltas
```

---

## MUSE Phases → Status After Integration

| Phase | Name | Tasks | What Exists | What's New | Priority |
|---|---|---|---|---|---|
| **P0** | Foundation | 8 | SessionGraph, PerformanceContext, conductor, CLI | .progression parser, .xmm parser (defer), L0-L3 compression | **Sprint 1** |
| **P1** | Harmonic Brain | 8 | `tension_arc.py`, chord generation (basic) | Voice leading, voicing engine, roman numeral analysis, enrichment | **Sprint 1** (highest value) |
| **P2** | Embeddings | 7 | `reference_analyzer.py` | ONNX runtime, vector index, semantic search | Sprint 4 |
| **P3** | Rhythm Engine | 8 | `humanize_physics.py`, `drum_humanizer.py`, `microtiming.py`, `groove_templates.py`, all performers | Genre timing tables, harmonic bass, arp transformation | **Sprint 2** |
| **P4** | Sound Oracle | 8 | `instrument_intelligence.py`, `preset_system.py`, `mix_chain.py` | Timbral ontology, frequency masking (partial in mix_engine) | Sprint 4+ |
| **P5** | Virtual MIDI | 8 | `mpc_exporter.py`, OSC bridge | rtmidi wrapper, loopMIDI detection, scheduling | **Sprint 3** |
| **P6** | Full Pipeline | 8 | `OfflineConductor`, `style_policy.py`, `arranger.py` | DAG orchestrator, stage integration, iteration loop | Sprint 3 |
| **P7** | Controller Intel | 7 | — | **DEFERRED** — MPC-specific, not critical path | Future |
| **P8** | Creative Frontier | 9 | `quality_validator.py`, `take_generator.py` | Multi-critic (VLC/BKAS/ADC), genre bending, creativity dials | **Sprint 2** |
| **P9** | Personalization | 8 | — | Preference tracking, producer modes, exploration | Sprint 4+ |
| **P10** | Live Performance | 8 | — | Real-time MIDI analyzer, harmonizer | Future |

---

## Contracts Adaptation

MUSE contracts (originally TypeScript) → Python dataclasses:

| Contract | MUSE Definition | Python Target | Adaptation |
|---|---|---|---|
| **C1** SongState | TypeScript interface | `SessionGraph` (already richer) | Add genre_dna, progression, L0-L3 methods |
| **C2** EnrichedProgression | TypeScript interface | `intelligence/harmonic_brain.py` | New dataclass |
| **C3** MidiPattern | TypeScript interface | `midi_generator.NoteEvent` (exists) | Already compatible |
| **C7** EmbeddingIndex | TypeScript interface | `intelligence/embeddings.py` | New module |
| **C8** GenreDNA | TypeScript interface | `intelligence/genre_dna.py` | New module (extends genre_intelligence.py) |
| **C9** SynthPresetCatalog | TypeScript interface | `preset_system.py` (exists) | Minor extension |
| **C10** EffectChain | TypeScript interface | `mix_chain.py` (exists) | Already implemented |
| **C11** ControllerMapping | TypeScript interface | Deferred | — |
| **C12** QualityScore | TypeScript interface | `quality_validator.py` (exists) | Extend with VLC, BKAS, ADC |
| **C13** ComposerPipeline | TypeScript interface | `conductor_offline.py` (exists) | Extend with DAG stages |
| **C14** UserPreferences | TypeScript interface | `intelligence/preferences.py` | New module |
| **C15** LivePerformanceState | TypeScript interface | Deferred (P10) | — |

---

## Quality Metrics (All Computable, No LLM)

| Metric | Acronym | What It Measures | Exists? | Sprint |
|---|---|---|---|---|
| Voice Leading Cost | VLC | Semitone movement per chord change | ❌ New | 1 |
| Rhythmic Consistency Score | RCS | Autocorrelation of pattern | ⚠️ Partial (in quality_validator) | 2 |
| Bass-Kick Alignment Score | BKAS | % of kicks with bass notes | ❌ New | 2 |
| Velocity Distribution Quality | VDQ | Stddev + hierarchy + no flatlines | ✅ In quality_validator | — |
| Arrangement Density Curve | ADC | Tension-density correlation | ❌ New | 2 |

---

## JUCE Strategy: Frozen Viewport

**Zero new JUCE features for 8 weeks.** Existing functionality is preserved:

- Piano roll, waveform, spectrum — passive display via OSC
- 16-voice synth — preview playback only
- Transport — syncs with Python pipeline
- FX chain — real-time monitoring

After Sprint 4: reassess JUCE investment based on whether copilot-Liku-cli chat becomes the primary interface.

---

## MPC Beats Data Extraction

Source files in `c:\dev\MPC Beats\`:

| Directory | Files | Format | Intelligence Value |
|---|---|---|---|
| `Progressions/` | 47 | `.progression` (JSON) | Professional voicings, genre-tagged harmony |
| `Arp Patterns/` | 80+ | `.mid` (Standard MIDI) | Rhythmic patterns, swing profiles, melodic contours |
| Synth plugins | 100+ | Directories | Preset names for recommendation (text-based) |

Sprint 1 parses Progressions + Arp Patterns into:
- `GenreHarmonicProfile` per genre (statistical fingerprints)
- `VoicingTemplate[]` (~750 real voicings from corpus)
- `RhythmProfile` per genre (density, swing, subdivision)

---

## Risk Mitigation

| Risk | Severity | Mitigation |
|---|---|---|
| Scope explosion (91 MUSE tasks + M1-M7 milestones) | CRITICAL | Only Sprint 1-2 intelligence modules. Everything else is Phase 2+ |
| Two-language maintenance (TypeScript specs → Python code) | HIGH | Treat specs as design docs. Implement Pythonically |
| LLM dependency for core function | HIGH | OfflineConductor as always-works fallback. LLM enhances, never required |
| JUCE distraction | MEDIUM | 8-week freeze. No C++ work until Sprint 4 reassessment |
| Untested MUSE theory | MEDIUM | Prototype each concept as isolated module, validate against MPC corpus |

---

## The Demo Moment (End of Sprint 3)

> User types in copilot-Liku-cli chat: **"dark neo-soul with gospel influences at 75 BPM"**
>
> Terminal streams:
> ```
> [Supervisor] Analyzing intent → genre: neo_soul+gospel, mood: dark, tempo: 75
> [Researcher] Genre DNA: [0.85, 0.72, 0.88, 0.65, 0.3, 0.55, 0.7, 0.4, 0.8, 0.85]
> [Builder]    Generating progression... Cm11 → Fm9 → AbMaj7#11 → Bb13sus4
> [Builder]    Voice-leading chords... VLC: 2.1 ✓
> [Builder]    Building bass line... chromatic approach, groove-locked
> [Builder]    Assembling drums... half-time, triplet hi-hats
> [Builder]    Arranging... intro(4) → verse(8) → chorus(8) → bridge(4) → outro(4)
> [Verifier]   Critics: VLC ✓ RCS ✓ BKAS ✓ VDQ ✓ ADC ✓ — PASSED
> [Builder]    Outputting → MPC Beats via virtual MIDI
> ```
>
> MPC Beats receives 4 tracks. Piano roll lights up.
> User assigns presets: TubeSynth 'Soul Keys', Bassline 'Round Sub', DrumSynth '808 Kit'.
> Presses play. **It sounds like music.**

---

## Reference Documents

| Document | Location | Purpose |
|---|---|---|
| Integration Synthesis Report | `MUSE/docs/muse-specs/MUSE-INTEGRATION-SYNTHESIS.md` | 4-reviewer consensus, architecture, algorithm diagnosis |
| Blueprint | `MUSE/docs/muse-specs/AI_MUSIC_PLATFORM_BLUEPRINT.md` | 921-line vision document |
| Orchestration State | `MUSE/docs/muse-specs/ORCHESTRATION_STATE.md` | 91 tasks, contract registry, decision log |
| Cross-Phase Contracts | `MUSE/docs/muse-specs/tasks/00-OVERVIEW-AND-CROSS-PHASE-CONTRACTS.md` | C1-C15 TypeScript interfaces |
| Task Decomposition P0-P2 | `MUSE/docs/muse-specs/tasks/TASK-DECOMPOSITION-P0-P2.md` | 23 tasks, C1-C8 detailed |
| Task Decomposition P3-P5 | `MUSE/docs/muse-specs/tasks/TASK-DECOMPOSITION-P3-P5.md` | 24 tasks, C9-C12 |
| Task Decomposition P6 | `MUSE/docs/muse-specs/tasks/TASK-DECOMPOSITION-P6.md` | 8 tasks, C13, DAG orchestrator |
| Task Decomposition P7-P10 | `MUSE/docs/muse-specs/tasks/TASK-DECOMPOSITION-P7-P10.md` | 32 tasks, C14-C15 |
| Agent Architecture Plan | `MUSE/docs/AGENT_ARCHITECTURE_PLAN.md` | Performer agent design (existing) |
| Remaining Tasks | `MUSE/remainingTasks.md` | Source of truth for existing backlog |

---

*This roadmap is grounded against the actual MUSE-ai codebase as of February 10, 2026. Every "exists" claim has been verified by reading the source files. Every "new" task references a specific MUSE spec.*
