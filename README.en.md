# Casual-Music

> A full-featured music production studio that runs entirely in your browser.

> `Casual-Music` is the project name; the repository identifier is `casual-music`.

**Status: requirements and architecture phase.** This repository currently contains the product, feature, and architecture specifications only. There is no runnable build yet.

---

## 1. Why this exists

Most browser-based music tools are deliberately simplified: a handful of loops, a few synths, a fixed set of presets, and a workflow that stops at "make a quick idea". They are fine for sketching, but they cannot replace a real production environment.

Casual-Music aims at the opposite end: a **browser DAW with desktop-class density**. Same workflow pillars as a professional studio — channel rack, step sequencer, piano roll, arrangement timeline, mixer, automation — but with a large built-in instrument and effect library, real project files, real export, and no install step.

Design principle: **complete the main creative path first, then widen the ecosystem.** A user should be able to go from empty project to a finished track without leaving the browser.

---

## 2. What it does

### Core workflow

| Area | Capability |
|---|---|
| Transport | Play/pause/stop, song and pattern modes, tempo with tap/half-time/fine-tune, time signature and swing, metronome, count-in, punch in/out, loop points |
| Channel Rack | Add/remove/reorder channels, per-channel volume/pan/mute/solo, routing, grouping and filtering, layering, channel settings |
| Step Sequencer | Up to 512 steps per pattern, per-step velocity and pitch, swing, step editing alongside piano roll data |
| Piano Roll | Note create/move/resize/slice/glue, velocity, pan, release, slide/portamento, ghost notes, note groups, quantize, humanize, snap-to-scale, chord stamping, event editor |
| Arrangement | Pattern, audio, and automation clips on a timeline, up to 500 tracks, multiple arrangements, performance mode, clip gain/fade/slice, markers and loop regions |
| Mixer | 500 insert tracks plus current and master, 10 effect slots per track, sends, sidechain, plugin delay compensation, EQ/phase/stereo/fader per channel, metering |
| Automation | Automation clips with curve and step modes, event automation, envelope/LFO/peak/XY/keyboard controllers, parameter linking, macros, automation recording and smoothing |
| Browser | Project content, samples, presets, plugin database, search, preview, tagging, favorites, missing-file relinking |
| Recording | Audio input recording with monitoring, MIDI recording and overdub, take management, integrated audio editor |
| Export | WAV 16/24/32-bit, FLAC, MP3, stem export, tail handling, LUFS metering, MIDI file export |

### Instruments

- **54 instrument plugins** covering every instrument category of a full desktop DAW: subtractive, FM, additive/resynthesis, physical modeling, samplers, drum machines, slicers, granular, wavetable, formant/speech, modular, rompler, solo acoustic, ensemble, chiptune, cinematic texture.
- **100+ instrument definitions** built on **18 shared DSP engine families**, so new instruments are a parameter schema plus a sample map, not a new engine.
- **General MIDI, 128 programs, all playable** out of the box, with a synthesis fallback so no program is silent.
- **60+ drum kits** spanning acoustic, electronic (808/909/707/CR-78/LinnDrum/DMX), hybrid, orchestral percussion, and world percussion.
- Full orchestral and world coverage: strings, brass, woodwinds, voices and choirs, guitars and plucked instruments, keys, mallet percussion, and regional instruments.

### Effects

**93 effects** across seven groups: EQ and filtering, dynamics, delay and time, reverb, modulation and spatial, distortion and saturation, utility and analysis. Flagship specs include an 8-band parametric EQ, a brickwall limiter with lookahead, a high-quality algorithmic reverb, convolution, a multiband maximizer, a time-warp engine, a vocoder, and a modular container.

Every effect supports parameter automation, presets, dry/wet, bypass, latency reporting for delay compensation, and stereo or sidechain operation where applicable.

---

## 3. Technical approach

| Decision | Choice | Rationale |
|---|---|---|
| Audio core | A single AudioWorklet DSP graph, not thousands of native Web Audio nodes | Sample-accurate scheduling, identical offline and realtime rendering, controllable CPU, easy automation and delay compensation |
| DSP language | TypeScript first, Rust/WebAssembly for heavy modules | Fast iteration where it matters, native-grade performance where it counts |
| UI | React plus Canvas2D, with custom-drawn and virtualized timeline and piano roll | 500 tracks and tens of thousands of notes at 60fps |
| State | Zustand with Immer plus a command bus, undo built from inverse commands | Predictable complex editing, clean path to collaborative editing |
| Project format | ZIP container with JSON manifest plus assets (own `.flw` format) | Versionable, portable, diff-friendly, collaboration-ready |
| Storage | Local-first with IndexedDB and OPFS, cloud sync later | No backend cost or privacy exposure in the early phases |
| Collaboration | Yjs CRDT over the same document model | Object-level merge instead of whole-file locking |

### Performance targets

| Metric | Target |
|---|---|
| Round-trip latency | ≤ 30 ms at 48 kHz / 128-frame buffer |
| Stability | 30 minutes continuous playback, no dropouts |
| CPU | 64 voices + 16 tracks + 8 effects under 60% of one core on a mid-range laptop |
| UI | 55–60 fps with 500 tracks, 10,000 notes, 1,000 clips |
| Cold start | ≤ 3 s to playable state, excluding sample decoding |
| Project load | 50 MB project in ≤ 3 s |
| Export | At least 2× realtime, cancellable, with visible progress |

### Browser support

Chrome and Edge are the primary targets; Firefox is supported; Safari 17+ is supported with documented degradations. AudioWorklet is a hard requirement.

---

## 4. Roadmap

| Phase | Scope | Exit criteria |
|---|---|---|
| 0 · Validation | AudioWorklet prototype, Canvas virtualization prototype, project I/O prototype, design tokens | Measured latency/CPU/frame-rate baselines; architecture decisions confirmed |
| 1 · MVP | Channel rack, step sequencer, piano roll basics, single arrangement, mixer with core effects, five instruments, automation basics, save/autosave/recovery, WAV export | A user can build and export a short track end to end |
| 2 · v0.5 | Advanced piano roll tools, multiple arrangements, performance mode, sends and sidechain, 30+ instruments, 25+ effects, recording, MIDI I/O, FLAC and MP3 export | Feature parity on the main creative path; audio regression suite green |
| 3 · v1.0 | Full instrument and effect library, GM 128, 60 drum kits, stem export, modular rack, controllers, lessons and templates, accessibility, offline PWA | All acceptance criteria and non-functional targets met |
| 4 · v1.x | Realtime collaboration, mobile and tablet layouts, assistant features, third-party WebAssembly plugin SDK, spatial audio, video sync | Incremental, non-blocking releases |

---

## 5. Non-goals for 1.0

- Hosting native VST/AU binaries (impossible in a browser sandbox; a WebAssembly plugin protocol is the long-term answer)
- Low-level audio drivers and inter-application audio bridging
- Pixel-identical UI, icons, or skins of any existing commercial DAW
- 100% fidelity bidirectional import of closed proprietary project formats
- Vendor account systems and first-party cloud sample subscriptions

---

## 6. Documentation

| Document | Contents |
|---|---|
| [`01-product-requirements.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/01-product-requirements.md) | Product goals, users, competitive positioning, non-functional requirements, acceptance criteria |
| [`02-feature-specification.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/02-feature-specification.md) | Per-window functional specification with requirement IDs, interaction rules, limits, and a full feature-surface audit |
| [`03-audio-and-technical-architecture.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/03-audio-and-technical-architecture.md) | Stack, audio engine, scheduling, rendering, storage, performance budgets, compatibility, security |
| [`04-instruments-catalog.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/04-instruments-catalog.md) | Engine families, instrument mapping tables, 133-plugin cross-check, GM 128 mapping, engine coverage matrix, sample pack strategy |
| [`05-effects-catalog.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/05-effects-catalog.md) | 93-effect catalog, flagship specifications, chain templates, priority tiers |
| [`06-data-model-and-project-format.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/06-data-model-and-project-format.md) | Entity schemas, `.flw` ZIP layout, version migration, import mapping |
| [`07-roadmap-risks-qa-compliance.md`](https://github.com/F1shLu0/Casual-Music/blob/dev/docs/fl-studio-web/07-roadmap-risks-qa-compliance.md) | Phased roadmap, effort estimates, risk register, QA strategy, sample licensing, export codec licensing |

The research archive lives in `references/raw/`: official manuals, plugin documentation, release notes, and technical papers used as the factual basis for these specifications.

---

## 7. Principles

1. **Original assets only.** No third-party trademarks, icons, factory presets, or bundled samples. Instruments and effects use our own names and our own sound design.
2. **Licensed content with provenance.** Every sample carries source, license, author, and hash. CC0 and CC BY material only, never non-commercial or no-derivatives assets. Each pack can be removed independently, and projects degrade gracefully to a "relink missing file" flow.
3. **Local-first by default.** Audio and project data stay on the device unless the user explicitly enables sync.
4. **Honest scope.** Features that a browser cannot deliver are listed as non-goals instead of being silently degraded.
5. **Accessibility is not optional.** Keyboard-completable core flows, WCAG AA contrast, 200% zoom without clipping.

---

## 8. Getting involved

The project is in specification review. The immediate next steps are: freeze the P0 scope, run the two technical spikes (audio engine and Canvas timeline), confirm sample and codec licensing, and stand up the project skeleton with CI, linting, and performance baselines.

If you want to contribute, start with the documentation above — the feature IDs and acceptance criteria are written to be directly convertible into issues and tests.