# MusicTeacher

**An open pedagogy engine for adaptive music education.**

MusicTeacher is a desktop application for music practice. It renders sheet music, plays it back, listens to you play it, and gives you structured, score-aware feedback — so you can practice an instrument without a teacher sitting next to you, and without a subscription.

It is built in C++ on [JUCE](https://juce.com) and the [Lomse](https://github.com/lenmus/lomse) music engraving engine.

> **Status (2026):** MusicTeacher is transitioning from a closed-source project (three years of solo development) to a fully open-source release. The full source code, build instructions, and binary releases for Linux, macOS, and Windows will be published here as the relicensing and cross-platform work lands. See [Roadmap](#roadmap) below.

---

## Why this exists

The open-source music software ecosystem covers two of the three layers a learner needs:

| Layer | Open-source state |
|---|---|
| **A. Notation engines** — render scores | Excellent: Lomse, VexFlow, LilyPond |
| **B. Score editors / DAWs** — author scores, play them back | Excellent: MuseScore, Frescobaldi, Ardour, LMMS |
| **C. Adaptive practice apps** — listen to the learner, give feedback | **Empty.** Every product in this category (Yousician, Simply Piano, Synthesia, Soundslice, Flowkey, Skoove) is closed, proprietary, and usually subscription-only |

MusicTeacher fills Layer C with a free, libre, offline-capable alternative.

---

## What it does (today, in the closed build)

- Loads and renders MusicXML scores using Lomse, with an OpenGL-accelerated drawing path
- Background-threaded layout with progressive chunk delivery (large scores stay responsive)
- Cached pre-rendered score buffers (`.raw`) and time-to-position maps (`.timemap`) for sub-frame-precise cursor synchronisation during playback
- MIDI input and audio playback
- Lesson library with structured exercises (currently bundled, English/Ukrainian)
- Settings UI: voices, MIDI configuration, accuracy thresholds, scoring, humanizer, GPU, language

## What's being added under NLnet funding

This project is the subject of an application to the [NLnet Foundation NGI Zero Commons Fund](https://nlnet.nl/commonsfund/), submitted in the June 2026 round. If funded, the work delivered over the following twelve months will include:

1. **License audit and FOSS migration** of the full codebase (target: GPLv3 for the application, MIT for the embeddable engine library)
2. **Cross-platform release** — Linux (ALSA / JACK / PipeWire), macOS (CoreAudio), Windows
3. **Adaptive Practice Engine** — a standalone, MIT-licensed C++ library performing real-time monophonic and polyphonic pitch tracking, onset detection, and rhythm scoring against a MusicXML target, with a stable C API consumable by any third-party music tool
4. **Open lesson library** — curated, CC-BY-SA progression of exercises (piano, guitar, voice) with structured difficulty metadata
5. **Upstream contributions to Lomse** — rendering performance patches, accessibility hooks, bug fixes
6. **Drafted skill-progression metadata schema** for MusicXML, submitted to the W3C Music Notation Community Group

If not funded, the open-source migration still happens — just on a slower, self-funded timeline.

---

## Roadmap

| Milestone | Target | Status |
|---|---|---|
| Public repository established | 2026-06 | ✅ this file |
| Dependency license audit complete | 2026-08 | ☐ |
| First public source release (Windows binary + buildable source) | 2026-10 | ☐ |
| Linux build (Flatpak) | 2026-12 | ☐ |
| macOS build (signed dmg) | 2027-01 | ☐ |
| Adaptive Practice Engine v0.1 (monophonic) | 2027-03 | ☐ |
| Adaptive Practice Engine v0.2 (polyphonic, chord recognition) | 2027-06 | ☐ |
| Open lesson library v1.0 | 2027-06 | ☐ |
| 1.0 release | 2027-09 | ☐ |

---

## Building from source

> Source is not yet public. Once the license audit and migration land (target 2026-10), this section will document the full build for each platform. Until then, here is the current development toolchain for reference:

- **Compiler**: MSVC 2022 (Windows) — Clang and GCC will be supported after the cross-platform port
- **Build system**: MSBuild (Visual Studio solution) — CMake migration is planned with the open release
- **Dependencies**: JUCE, Lomse, FreeType, libpng, zlib, bzip2 (via vcpkg on Windows)
- **Entry point**: `Source/Main.cpp`, `Source/MainComponent.h`

---

## Architecture (high level)

```
┌──────────────────────────────────────────────────────────────────┐
│                         MusicTeacher                              │
├──────────────────────────────────────────────────────────────────┤
│  UI (JUCE Components)                                             │
│    SettingsDialog, ScoreView, lesson playback controls            │
├──────────────────────────────────────────────────────────────────┤
│  Score subsystem                                                  │
│    ScoreViewLoadingMethods    BackgroundLomseRenderer             │
│    PreRenderManager (cache)   ScoreViewRenderingMethods (paint)   │
│    CursorManager (uses TimeToXMap directly)                       │
├──────────────────────────────────────────────────────────────────┤
│  Lomse engraving engine (LGPL, external)                          │
├──────────────────────────────────────────────────────────────────┤
│  Audio + MIDI (JUCE abstraction over WASAPI/ASIO/CoreAudio/ALSA)  │
├──────────────────────────────────────────────────────────────────┤
│  [Coming] Adaptive Practice Engine                                │
│    pitch tracker · onset detector · score-aligned scorer          │
└──────────────────────────────────────────────────────────────────┘
```

Two design notes worth knowing:

- **Lomse's graphic model is lazy.** `new_document()` is cheap; layout only happens on the first access to the graphic model. We do that first access on a background thread and stream the resulting render into a cache, so the UI thread never blocks on Lomse layout.
- **Playback cursor is decoupled from the graphic model.** Once the `TimeToXMap` is built and the pre-rendered buffer is cached, cursor tracking during playback no longer needs the live Lomse graphic model. This is what makes the cursor responsive on large scores.

A full architecture document will be published with the first source release.

---

## License

The full project will be released under **GPLv3**, with the standalone Adaptive Practice Engine library released under **MIT** so it can be embedded by other music software regardless of their license.

(Final license selection pending the dependency audit. If a transitive dependency forces a different choice, this README will be updated accordingly.)

---

## Contributing

Source is not yet open, so we are not yet accepting code contributions. The fastest ways to help right now:

- **Star and watch** the repository to signal interest — this materially helps the NLnet application
- **Open an issue** if you are a music teacher and want to describe a feature that would make this useful in your lessons — pedagogical input is the rarest and most valuable kind of contribution
- **Reach out** if you are interested in being a music-domain consultant on the funded work (piano, guitar, voice pedagogy especially)

Contact: musicneutrino@gmail.com

---

## Acknowledgments

- [Lomse](https://github.com/lenmus/lomse) by Cecilio Salmeron — the music engraving engine MusicTeacher renders on top of
- [JUCE](https://juce.com) — the C++ application framework
- The W3C Music Notation Community Group for [MusicXML](https://www.musicxml.com/)
- (Pending) NLnet Foundation and the European Commission's Next Generation Internet initiative

---

*MusicTeacher is developed in Ukraine.*
