# INSTRUCTIONS.md 
# — Build the Ultimate Multimodal AI Music Generator (Text-to-MIDI-to-Audio for Any DAW/Sequencer)

You are now tasked with building **multimodal-ai-music-gen**, a production-ready, open-source Python-based system that takes any natural language text prompt (e.g., "a dark trap soul beat at 87 BPM in C minor with sidechained 808, moody piano chords, and vinyl crackle") and generates a complete, standalone music project. The output includes MIDI files for symbolic music (notes, velocities, timing), rendered WAV audio files (for playback), and an optional export to Akai MPC .xpj format (inspired by the "avocado" project structure for seamless integration with MPC Software 2.13.1.27+).

This system leverages multimodal AI: text processing for prompt parsing, deep learning for music generation (MIDI via transformers/GANs, audio via diffusion models), and file export for production use. It must be offline-capable where possible, using pre-trained models, and royalty-free by default (generate or bundle basic samples).

Grounded in:
- **Akai MPC Docs** (from official support & forums): .xpj is XML-based (no namespace in base, but expandable); requires sibling `[ProjectName]_[ProjectData]` folder for .wav/.xpm/.xal assets; sequences use 480 PPQ ticks; programs (.xpm) map pads/samples.
- **Python MIDI Libs** (mido, pretty_midi, MIDIUtil): For humanized sequences (velocity 70-127 variation, swing via tick offsets).
- **Audio Gen** (numpy, scipy, wave, soundfile): Procedural WAV synthesis; optional Magenta/TensorFlow for advanced.
- **AI Music Tutorials** (Magenta, Text2Midi, MusicGen): Text-to-MIDI via transformers; MIDI-to-audio via FluidSynth or diffusion.
- **Project Structure** (from MPC Tutor & Akai): .xpj + [ProjectData] for assets; ensure relative paths like `.\[ProjectData]\kick.wav`.

Target: VS Code project, Python 3.12+, pip-installable. Generate 2-3 min tracks with arrangement (intro/drop/breakdown/outro).

## CORE REQUIREMENTS (NON-NEGOTIABLE)

1. **Input**: Text prompt parsed for genre, BPM (60-180), key (e.g., C minor), elements (drums, chords, textures like vinyl crackle).
2. **Output Formats**:
   - **MIDI**: Multi-track .mid (drums, bass, melody) – humanized (random vel/timing ±10%).
   - **Audio**: Rendered .wav stems (44.1kHz/16-bit) + full mix.
   - **MPC Export (Optional Flag)**: .xpj + `[ProjectName]_[ProjectData]` folder with .xpm kits, .xal sequences, .wav assets (relative paths only; no absolute).
3. **Offline-First**: Use pre-trained models (e.g., Hugging Face Diffusers for AudioLDM2); fallback to procedural gen (sine waves for tones, noise for crackle).
4. **Multimodal**: Text → Embeddings (via transformers) → MIDI tokens → Audio synthesis.
5. **Avocado Tie-In**: Default example prompt: "lofi hiphop beat with rhodes and rain at 87bpm in C minor" – outputs MPC-compatible project with 8 tracks (drums, 808, rhodes, piano, strings, vinyl, vocal chop, texture).

## PROJECT STRUCTURE (Create This Exactly in VS Code)

```
multimodal-ai-music-gen/
├── main.py                      # CLI entry: python main.py "prompt" [--mpc] [--stems]
├── multimodal_gen/
│   ├── __init__.py
│   ├── prompt_parser.py         # NLP: Extract BPM/key/genre/elements (use transformers or regex)
│   ├── midi_generator.py        # Text-to-MIDI: Use MusicVAE/GAN (mido + pretty_midi for humanization)
│   ├── audio_renderer.py        # MIDI-to-WAV: FluidSynth or numpy synthesis (wave/scipy for export)
│   ├── mpc_exporter.py          # .xpj/.xpm/.xal builder (xml.etree; relative paths)
│   ├── arranger.py              # Song structure: Intro (4 bars light) → Drop → Variation → Breakdown → Outro
│   ├── assets_gen.py            # Procedural samples: Sine for notes, noise for crackle (numpy/wave)
│   └── utils.py                 # Helpers: UUIDs, ticks (480 PPQ), royalty-free defaults
├── models/                      # Pre-trained (download on init): MusicGen, AudioLDM2 weights
├── assets/
│   ├── templates/               # empty.xpj, default.xpm (drum kit XML)
│   └── samples/                 # Bundled royalty-free: basic_808.wav, vinyl_crackle.wav (generate if missing)
├── output/                      # Generated projects (e.g., avocado_[ProjectData]/)
├── requirements.txt             # torch, transformers, mido, pretty_midi, soundfile, scipy, xml.etree, diffusers
└── README.md                    # Usage, examples (incl. avocado prompt)
```

## EXACT FORMATS & SCHEMAS (From Grounded Sources)

### MIDI (.mid – mido/MIDIUtil)
- Tracks: 0 (meta/tempo), 1 (drums: ch10), 2 (bass), 3 (chords/melody).
- Humanize: Vel random(80-120); timing offset ±5% for swing.
- Example: 87 BPM → tempo=87; 4/4, 480 ticks/beat.

```python
from mido import MidiFile, MidiTrack, Message
mid = MidiFile(ticks_per_beat=480)
track = MidiTrack()
mid.tracks.append(track)
track.append(Message('program_change', program=0, time=0))  # Piano
for note, time, dur, vel in notes:  # e.g., [(60, 0, 480, 100)]
    track.append(Message('note_on', note=note, velocity=vel, time=time))
    track.append(Message('note_off', note=note, velocity=0, time=dur))
mid.save('output.mid')
```

### Audio (.wav – numpy/wave/soundfile)
- 44.1kHz, 16-bit, mono/stereo.
- Procedural: Sine for tones (440Hz A4); noise for textures.
- Render: Use midi2audio/FluidSynth for MIDI→WAV.

```python
import numpy as np
import soundfile as sf
sample_rate = 44100
t = np.linspace(0, duration, int(sample_rate * duration), False)
note = 440 * np.sin(2 * np.pi * freq * t)  # Sine wave
sf.write('note.wav', note, sample_rate)
```

### MPC Export (.xpj + [ProjectData] – xml.etree)
- .xpj: XML root `<Project>`; `<BPM>87.000000</BPM>`; `<Tracks>` with `<Track name="Drums" type="Drum">`; `<Sequence number="1">` with `<Events><Note pitch="36" start="0" duration="480" velocity="100"/></Events>`.
- [ProjectData]: .wav assets, .xpm (drum programs: `<Program><Pad note="36"><Sample>kick.wav</Sample></Pad></Program>`), .xal (sequences/songs).
- Relative paths: `<File>.\avocado_[ProjectData]\kick.wav</File>`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Project>
  <BPM>87.000000</BPM>
  <Key>C Minor</Key>
  <FileList><File>.\avocado_[ProjectData]\kick.wav</File></FileList>
  <Tracks>
    <Track name="Drums" type="Drum">
      <Program>.\avocado_[ProjectData]\drums.xpm</Program>
      <Sequence number="1"><Events><Note pitch="36" start="0" duration="480" velocity="100"/></Events></Sequence>
    </Track>
  </Tracks>
  <Song><Sequence number="1" bars="16"/></Song>  <!-- Arrangement -->
</Project>
```

## REQUIRED FEATURES (All Must Work – Digestible Sections)

### Section 1: Prompt Parsing (prompt_parser.py)
- Use regex/transformers: Extract BPM, key, genre (e.g., "trap soul" → minor scale, 808 focus).
- Elements: Drums (generate kit), chords (Rhodes/piano), textures (vinyl → noise gen).
- Default: Avocado – 87 BPM, C minor, lofi/trap soul.

### Section 2: MIDI Generation (midi_generator.py)
- Base: Procedural (MIDIUtil) or AI (Magenta MusicVAE for drums; Text2Midi transformer for melody).
- Humanize: Add ±5-10% timing/vel variation; swing (tick offset 0.55*480).
- Tracks: Drums (GrooVAE-style grooves), bass (808 slides), melody/chords (pentatonic in key).
- Arrangement: 2-3 min (e.g., 4-bar loop x4 for drop; vary seq2 for breakdown).

### Section 3: Audio Rendering (audio_renderer.py)
- MIDI→WAV: midi2audio + FluidSynth (bundle free soundfont).
- Procedural Fallback: Numpy sine/percussion synthesis; add effects (reverb via scipy, sidechain ducking).
- Stems: Export per-track .wav; mix via numpy add (normalize -1 to 1).
- Textures: Vinyl crackle (white noise low-pass); rain (filtered noise).

### Section 4: MPC Exporter (mpc_exporter.py – Avocado Focus)
- Build .xpj XML (from template; insert BPM/key/sequences).
- .xpm Kits: XML pads mapping .wav (e.g., pad A01=note36=kick).
- [ProjectData]: Copy/ generate .wav, .xpm, .xal (song XML).
- Ensure: Relative paths; UUIDs for tracks; 480 ticks/beat.

### Section 5: Song Arranger (arranger.py)
- Structure: Parse prompt for sections (e.g., "switch-up" → seq3).
- Default Avocado: Intro (vinyl fade), Drop (full groove), Variation (add piano), Breakdown (pad swell), Outro (fade).

### Section 6: One-Command Usage (main.py)
```bash
python main.py "dark trap soul at 140 BPM in F minor with 808 and vocal chops" --mpc --stems
```
- Outputs: `output.mid`, `output.wav`, `output_[ProjectData]/output.xpj`.
- Interactive: `--interactive` → Query BPM/key.

## BONUS FEATURES (Include for Premium Polish)
- **AI Enhancements**: Integrate MusicGen (Audiocraft) for text-to-audio direct; fine-tune on MIDI datasets.
- **Royalty-Free Defaults**: Bundle 10 basic .wav (808, hats, rhodes chord loop) – generate procedurally if missing.
- **Export Options**: `--stems` (per-track WAV); `--dark-mode` (minor keys, low-pass filters).
- **Testing**: Include avocado example: Run prompt → verify loads in MPC (no errors).

## FINAL DELIVERABLES
1. Complete folder structure with all .py files (tested: `python main.py "avocado prompt" --mpc` generates loadable .xpj).
2. `requirements.txt`: Minimal deps (mido, numpy, scipy, torch, transformers, soundfile, pretty_midi, midi2audio).
3. Avocado Example Output: Full .xpj snippet + [ProjectData] description.
4. README: Install (`pip install -r requirements.txt`), usage, troubleshooting (e.g., "missing fonts? Download FluidR3_GM.sf2").

This builds the future of multimodal music: Text → AI → Playable Art. Begin coding now – aim for offline-first, MPC-compatible by default. Output zipped project or commit-ready files.