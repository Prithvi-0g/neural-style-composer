# Neural Style Composer
### Hack The Arts — Devpost Hackathon (deadline: August 1, 2026)

> "Create art that couldn't exist without technology."

A browser-based painting canvas that analyzes pixel data in real time and composes music from what it sees. No AI APIs — a custom pixel analysis engine reads color, texture, and composition directly and maps them to musical parameters, which drive a Tone.js synthesizer.

---

## Current State

**Phase 3 (Execution) — Core loop is complete and working.**

The single deliverable file is `neural-style-composer.html` — a fully self-contained HTML/CSS/JS app with no build step. Open it in a browser and it works.

What's done:
- ✅ Full painting canvas (brush, eraser, fill, undo, clear)
- ✅ Brush size + opacity controls, 12-color palette + custom color picker
- ✅ Custom pixel analysis engine (no API, pure JS)
- ✅ Tone.js music engine with sequencer, reverb, optional bass layer
- ✅ Waveform visualizer overlay on canvas
- ✅ Quadrant zone display (4-cell grid showing per-zone dominant color)
- ✅ Auto-reanalysis every 6s while playing — music shifts as you paint more
- ✅ Export PNG

---

## Architecture

Single-file app. All logic lives in `<script>` inside `neural-style-composer.html`.

### Pixel Analysis Engine (`analyzeCanvas()`)

Runs on the raw canvas `ImageData`. No external calls.

| Function | What it does |
|---|---|
| `samplePixels(step=6)` | Downsamples canvas on a 6px grid, skips background pixels, converts each to HSL |
| `analyzeQuadrants(samples)` | Splits canvas into TL/TR/BL/BR zones, computes avg hue/sat/lit per zone |
| `measureEdgeDensity()` | Counts sharp pixel transitions → detects stroke roughness vs smooth washes |
| `measureColorVariance(samples)` | Hue variance across all samples → harmonic complexity |
| `hueToKey(hue, minor)` | Maps avg hue to musical key via circle of fifths spread over 360° |
| `pickScale(variance, sat)` | Low variance → pentatonic; high → chromatic/dorian |
| `pickInstruments(hue, lit, edgeDensity)` | Hue range + texture → instrument timbre |
| `pickMood(hue, sat, lit, edgeDensity)` | Returns a single mood word for the UI |

**Output object** from `analyzeCanvas()`:
```js
{
  tempo,        // 60–180 BPM
  key,          // e.g. "D minor"
  scale,        // pentatonic | major | minor | dorian | chromatic
  reverb,       // 0.0–1.0
  density,      // 0.0–1.0 (note frequency)
  instruments,  // string[] up to 3
  mood,         // string
  avgH, avgS, avgL,   // canvas-wide HSL averages
  edgeDen,      // edge density scalar
  colorVar,     // color variance scalar
  coverage,     // fraction of canvas painted
  quads,        // [{hue,sat,lit,count}] × 4
}
```

### Music Engine (Tone.js 14.8.49)

| Function | What it does |
|---|---|
| `startMusic(params)` | Tears down previous audio graph, builds new synth + reverb, starts 16-step Tone.Sequence |
| `stopMusic()` | Disposes all audio nodes in `audioNodes[]`, stops transport |
| `teardownAudio()` | Called by both — cleans up loopRef, audioNodes[], transport |
| `getScaleNotes(key, scale, octaves)` | Returns array of Tone.js note strings for the given key+scale |
| `animateWave(params)` | rAF loop drawing sine wave on `#waveCanvas` overlay |

**Key fix applied**: `Tone.Sequence` must receive a real array (e.g. `[0..15]`) as its second argument, not `null`. Also `Tone.Reverb` requires `.generate()` after construction. All audio nodes are pushed to `audioNodes[]` so nothing gets GC'd.

---

## What Needs to Be Done Next

### Priority 1 — Polish & UX
- [ ] Smoother musical transitions when auto-reanalysis updates params (currently restarts the sequencer; should instead ramp BPM and morph synth params without hard restart)
- [ ] Add a volume master control to the panel
- [ ] Keyboard shortcuts: `Space` = play/stop, `Cmd+Z` = undo, `E` = eraser, `B` = brush
- [ ] Show a small "analyzing…" pulse animation when auto-reanalysis fires

### Priority 2 — Musical Quality
- [ ] The scale/key mapping could be refined — currently pure hue-to-key, could weight more by saturation
- [ ] Add a second melody voice that plays counterpoint to the main sequence
- [ ] Chord mode: on low-density paintings, trigger full chords instead of single notes
- [ ] Arpeggiation pattern variety (currently always straight 16th-note grid)

### Priority 3 — Submission Requirements
- [ ] Record a compelling 2–3 min demo video showing a painting session with music evolving
- [ ] Deploy to GitHub Pages (just push the single HTML file)
- [ ] Write Devpost project description (Phase 4 — do this in Claude.ai chat, not here)
- [ ] Add attribution comments in HTML for Tone.js and Google Fonts

### Stretch Goals
- [ ] MIDI export — let users download their composition as a MIDI file
- [ ] "Score" text export — download a human-readable description of what the algorithm detected
- [ ] Multiple canvases / layers
- [ ] Webcam mode — analyze a live camera feed instead of a painted canvas

---

## Hackathon Judging Rubric

| Criterion | Weight | Our angle |
|---|---|---|
| Creativity & Originality | 30% | Synesthesia (painting → music) via custom pixel engine — not a wrapper around an existing tool |
| Use of Technology | 25% | Custom pixel analysis algorithm; Tone.js synthesis; real-time quadrant analysis; no API dependency |
| Interactivity & Experience | 20% | Music changes live as you paint; waveform visualizer; zone display reacts to painting |
| Execution | 15% | Single-file, no build step, works offline |
| Theme Alignment | 10% | Art that literally could not exist without computation — the music is the mathematical output of the visual |

---

## Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| Canvas / UI | Vanilla HTML5 Canvas + CSS Grid | No framework |
| Pixel Analysis | Custom JS | `analyzeCanvas()` and helpers |
| Music | Tone.js 14.8.49 (CDN) | PolySynth, Reverb, Sequence, Transport |
| Fonts | Google Fonts (Space Mono + Inter) | CDN |
| Hosting (target) | GitHub Pages | Push single HTML file |

---

## How to Run

```bash
# No install needed. Just open the file:
open neural-style-composer.html

# Or serve locally if you hit any CORS issues:
python3 -m http.server 8080
# then visit http://localhost:8080/neural-style-composer.html
```

---

## Session History (for context)

This project was scoped and built across a Claude.ai chat session (Hack The Arts hackathon strategy guide). The ideation phase produced 5 ideas; this project (originally "Neural Style Composer — AI vision pipeline") was selected and pivoted to a custom pixel analysis engine instead of calling an API, which strengthens the "Use of Technology" judging score significantly.

The other 4 ideas in reserve (if a pivot is needed):
1. **Bioelectric Canvas** — EMG sensor → generative art (highest ceiling, hardware required)
2. **Quantum Noise Painter** — hardware true-RNG seeding generative visuals
3. **Echolocation Art** — ultrasonic sensor array → visual/audio output
4. **Living Typography** — cellular automata applied to typed text
