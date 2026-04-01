# 🎧 quitebeyond's Spatial Audio Web Player

A browser-based 3D spatial audio workstation for binaural mixing, HRTF rendering, and immersive soundscape design. Single HTML file — no build step, no server required.

## Features

- **Studio Mode** — Fixed speaker layouts (Stereo, 5.1, 7.1, 7.1.4 Atmos) with ITU-standard positioning
- **Soundscape Mode** — Dynamic sound sources with free 3D placement, split-aware L/R stereo handling
- **SOFA/HRTF Rendering** — Load measured HRTF datasets (.sofa) for accurate binaural spatialization via KU100 or custom dummy heads
- **Real-time 3D Visualization** — Interactive Three.js scene with draggable speakers, listener dummy, room dimensions
- **Binaural Export** — Render to 24-bit WAV or WebM/Opus with full spatial processing
- **Room Simulation** — Algorithmic reverb via ConvolverNode with RT60-based impulse response generation
- **VBAP** — Vector Base Amplitude Panning for multichannel output (5.1/7.1)
- **Formations & Randomize** — Ring, Dome, Front Arc layouts with mirror-aware / split-aware positioning
- **Session Persistence** — Auto-save/restore via localStorage

## Libraries & Dependencies

### 3D Rendering

| Library | Version | CDN | Purpose |
|---------|---------|-----|---------|
| [Three.js](https://threejs.org/) | r128 | [cdnjs](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js) | 3D scene, speaker/listener objects, raycasting, camera controls |

### Audio

| Library | Version | CDN | Purpose |
|---------|---------|-----|---------|
| [Resonance Audio](https://resonance-audio.github.io/resonance-audio/) | latest | [jsDelivr](https://cdn.jsdelivr.net/npm/resonance-audio/build/resonance-audio.min.js) | Room acoustics model (optional fallback — primary reverb uses custom ConvolverNode IR) |

### SOFA / HDF5 Parsing

| Library | Version | CDN | Purpose |
|---------|---------|-----|---------|
| [h5wasm](https://github.com/usnistgov/h5wasm) | 0.7.3 | [jsDelivr](https://cdn.jsdelivr.net/npm/h5wasm@0.7.3/dist/esm/hdf5_hl.js) | Read .sofa files (HDF5 format) in-browser — extracts HRTF impulse responses, source positions, sample rates |

### Fonts

| Font | Weights | Source | Usage |
|------|---------|--------|-------|
| [Space Mono](https://fonts.google.com/specimen/Space+Mono) | 400, 700 | Google Fonts | UI labels, values, monospace elements |
| [Syne](https://fonts.google.com/specimen/Syne) | 400, 600, 800 | Google Fonts | Headings, body text |

## Web APIs Used

No external audio processing libraries — all spatial audio runs on native browser APIs:

| API | Usage |
|-----|-------|
| **Web Audio API** | Complete audio graph: `AudioContext`, `PannerNode` (HRTF), `ConvolverNode` (SOFA IRs + room reverb), `GainNode`, `AnalyserNode`, `BiquadFilterNode`, `ChannelSplitter/Merger`, `OfflineAudioContext` (export rendering) |
| **AudioParam Automation** | `setValueAtTime`, `linearRampToValueAtTime`, `setTargetAtTime`, `setValueCurveAtTime` — for crossfades, gain scheduling, reverb fade-in |
| **MediaRecorder API** | WebM/Opus binaural export (Chrome 94+) |
| **File API / FileReader** | Audio file loading, SOFA import, header-based stereo detection (WAV/FLAC/MP3/OGG channel count from first 512 bytes) |
| **localStorage** | Session persistence (positions, volumes, mute/loop states, room dimensions, HRTF preferences) |
| **Drag and Drop API** | File import via drag-and-drop zones |

## Audio Algorithms (Custom Implementation)

These are implemented from scratch — no libraries:

| Algorithm | Description |
|-----------|-------------|
| **VBAP (2D)** | Vector Base Amplitude Panning for multichannel speaker routing |
| **KU100 SOFA Renderer** | Nearest-neighbour IR lookup from SOFA datasets, per-channel ConvolverNode with crossfade restart |
| **Room IR Generator** | Synthetic impulse response from room dimensions + RT60 — exponential decay with frequency-dependent absorption |
| **Distance Attenuation** | Inverse distance model matching Web Audio spec: `gain = refDist / (refDist + rolloff × (dist - refDist))` |
| **Stereo Split** | Real-time L/R channel extraction via ChannelSplitter + per-channel GainNodes — no re-encoding |
| **Audio Header Parser** | Lightweight channel count detection from file headers (WAV byte 22, FLAC STREAMINFO, MP3 frame header, OGG/Vorbis+Opus identification) |
| **Crossfade Engine** | 20ms equal-power crossfade for seamless SOFA IR swaps during speaker movement |

## Browser Compatibility

| Browser | Support | Notes |
|---------|---------|-------|
| Chrome 94+ | ✅ Full | Recommended — Opus export, full Web Audio support |
| Firefox 100+ | ✅ Full | WebM/Opus export may use different codec |
| Safari 15.4+ | ⚠️ Partial | No Opus export, HRTF PannerNode quality varies |
| Edge 94+ | ✅ Full | Chromium-based, same as Chrome |

## Architecture

Single-file HTML application (~12,900 lines). No build tools, no bundler, no framework.

```
┌─────────────────────────────────────────────────┐
│  HTML/CSS UI (Sidebars, Modals, Player)         │
├─────────────────────────────────────────────────┤
│  Three.js 3D Scene                              │
│  ├─ Room (BoxGeometry + Grid)                   │
│  ├─ Speakers (draggable groups)                 │
│  ├─ Listener Dummy (head + ear markers)         │
│  └─ Particle system (note animations)           │
├─────────────────────────────────────────────────┤
│  Web Audio Graph                                │
│  ├─ Source → ExtractNodes → GainNode            │
│  ├─ → PannerNode (Browser HRTF)                │
│  │   OR ConvolverNode (SOFA/KU100)              │
│  │   OR ChannelMerger (VBAP)                    │
│  ├─ → MasterGain → destination                  │
│  └─ → SendBus → Convolver (Room IR) → WetGain  │
├─────────────────────────────────────────────────┤
│  State Management                               │
│  ├─ StateStore (reactive, event-driven)         │
│  ├─ SessionManager (localStorage persistence)   │
│  └─ EventBus (pub/sub for decoupled modules)    │
└─────────────────────────────────────────────────┘
```

## Quick Start

1. Download `player_v10.html`
2. Open in Chrome/Firefox
3. Drop audio files on the left zone, a .sofa file on the right zone
4. Press Space or ▶ to play

No server needed — everything runs client-side.

## License

Copyright (C) 2024-2026 **quitebeyond**

This program is free software: you can redistribute it and/or modify it under the terms of the **GNU General Public License v3.0** as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but **WITHOUT ANY WARRANTY**; without even the implied warranty of **MERCHANTABILITY** or **FITNESS FOR A PARTICULAR PURPOSE**. See the [GNU General Public License](https://www.gnu.org/licenses/gpl-3.0.txt) for more details.

Third-party libraries (Three.js, Resonance Audio, h5wasm, Google Fonts) retain their original licenses — see [LICENSE](LICENSE) for details.

### What this means

| You may | You must | You may not |
|---------|----------|-------------|
| Use commercially | Distribute source code with any copy | Sublicense under a different license |
| Modify freely | Keep the copyright notice & author name | Remove attribution to quitebeyond |
| Distribute copies | License modifications under GPL-3.0 | Hold the author liable for damages |
| Use privately | State significant changes made | Use the name "quitebeyond" for endorsement |

