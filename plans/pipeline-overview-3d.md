# Plan: `animations/haidis-pipeline-overview-3d.html` — 3D Adaptive Pipeline Animation

## Context

`animations/haidis-training-phase-3d.html` is a Three.js remake of the SVG `training-phase.html`. This plan builds the 3D counterpart of `pipeline-overview.html`: the traditional processing pipeline (DAQ → Raw → Cal & Reco → Intermediate → Additional Analysis → AI-Ready) feeding HAIDIS + SAGIPS. Over three phases, HAIDIS moves upstream until it trains directly on raw detector data, removing the processing steps after that point.

The 3D version tells the same story with the same timing as the 2D version. It also uses things a flat SVG can't do well:
- **Bypassed stages sink into the floor** and leave a dashed footprint. This replaces the 2D "fade to 12 % opacity" ghosts, which would overlap badly in 3D.
- **HAIDIS + SAGIPS ride a hover-sled** that glides upstream along the pipeline axis. This replaces the 2D CSS `x` slide.
- **Stage height and connector flow speed scale with data rate**, so data reduction along the pipeline is visible in the shapes themselves.
- **The SAGIPS inference histogram grows as 3D bars** on a holographic panel above the model.

The 2D `pipeline-overview.html` is not modified.

---

## File to create

`animations/haidis-pipeline-overview-3d.html`. Use a clean `<!DOCTYPE html><html lang="en"><head>…</head><body>` structure. The training-3D file has an exported-artifact wrapper with `<title>` inside `<body>`, so don't copy that.

---

## Conventions inherited from `haidis-training-phase-3d.html` (not the SVG conventions in CLAUDE.md)

| Concern | Convention to copy |
|---|---|
| Stack | Three.js **r128** from cdnjs, single self-contained file, no GSAP |
| Theme | Dark "instrument room": `--bg #0c1524`, `--panel`, `--line`, `--fg`, `--muted`; fonts Barlow Semi Condensed + IBM Plex Mono |
| Scene | Fog, hemisphere + shadowed directional light, floor plane + `GridHelper`, translucent floor strips |
| Helpers | `glow()`, `std()`, `basic()`, `tube()`, `dashed()`, `arc()`, `FLOW_TEX` scrolling-gradient texture, canvas-texture builders |
| Labels | HTML `.lbl` readouts projected from 3D anchors each frame (`addLabel` / `placeLabels`), `.sec` lines hidden < 720 px |
| HUD | Eyebrow / h1 / sub top-left; view buttons + Pause + SVG legend top-right; hint + `t=` clock bottom-right |
| Camera | Landscape/portrait presets, drag-orbit, wheel/pinch zoom, 1.2 s tween, `userMoved` respected on resize |
| Time | Pausable sim clock `simT` with `after()` queue, `move()` curve movers, `step(dt)` / `decay(dt)` split |
| Motion | `prefers-reduced-motion` → start paused, instant camera moves |

### Palette (semantic tokens, same hex values as training-3D)
| Element | Token | Value | 2D equivalent |
|---|---|---|---|
| DAQ / Raw Detector Data | `--raw` (**new**) | `#ee4b5c` (brightened `#cc3344`; tune for contrast) | `#cc3344` |
| Calibration & Reco | `--cal` | `#ff7a3d` | `#e05820` |
| Intermediate Data | `--int` | `#e2b13c` | `#c89000` |
| Additional Analysis | `--ana` | `#7ea2d6` | `#4a70a0` |
| AI-Ready Data | `--ai` | `#2fd48a` | `#1a9060` |
| HAIDIS Training | `--haidis` | `#b65ae6` | `#9933bb` |
| SAGIPS Model | `--sagips` | `#22b8e8` | `#0099cc` |
| Histogram bars | `--buf` | `#5b8fd0` | `#4a7ab5` |
| Pipeline connectors | `--muted` | `#8ea3c2` | `#8aaac8` |

Phase colors = source colors: Phase 1 `--ai`, Phase 2 `--int`, Phase 3 `--raw`.

---

## Scene layout (world units, +x = downstream, +z = toward default camera)

Shape scale constant `K = 1/50` converts 2D SVG polygons to 3D footprints so silhouettes match the original.

| Element | x (center) | Footprint x × z | Height | Form | Token |
|---|---|---|---|---|---|
| DAQ / Detector | −23.5 | torus r = 2.0 | ring at y .5 on 4 posts | flat torus, 4 detector modules (N/S/E/W), beam-pipe cross, interaction-point sphere | `--raw` |
| Raw Detector Data | −17.5 | 2.2 × 2.0 | 2.4 | box; front face = scrolling bit-noise canvas texture | `--raw` |
| Calibration & Reco | −12.5 | 2.56 × 2.0 | 2.0 | extruded chevron (`268,355 378,355 396,405 378,455 268,455` × K) | `--cal` |
| Intermediate Data | −7.5 | Ø 2.0 | 1.6 | 3-disc DB stack with emissive rims (like the MLFlow stack) | `--int` |
| Additional Analysis | −2.5 | 2.56 × 2.0 | 1.3 | extruded chevron | `--ana` |
| AI-Ready Data | 2.5 | 2.4 × 2.0 | 1.0 | box + 3-2-1 sphere neural-net glyph on top | `--ai` |
| HAIDIS Training (sled origin) | `sledX(phase)` | 4.3 × 2.2 | 0.9, hovering at y .25 | extruded notched chevron (`900,350 1090,350 1115,405 1090,460 900,460 920,405` × K) | `--haidis` |
| SAGIPS Model (sled-local +5.0) | sled + 5.0 | 3.1 × 2.2 | 1.6 | dark glass box, cyan edges, spinning wireframe icosahedron core + glow | `--sagips` |
| Plot panel (sled-local) | sled + 5.0, y 3.0, z −0.6 | 5.2 × 3.2 panel | — | yaw-billboarded holographic panel with 40 bar meshes | `--buf` |
| Theory Update (sled-local) | sled + 5.0, z +2.4 | 3.0 × 1.06 | upright | upward chevron (`1143,530 1143,500 1218,477 1293,500 1293,530` × K) + canvas text | `--sagips` |
| Data-rate ruler (floor) | −17.5 → 2.5, z +3.2 | tapered strip 1.6 → 0.2 wide | floor | ticks `~10–100 Gbps` / `~1–10 Gbps` / `< 1 Gbps` under Raw / Int / AI | gradient `--raw`→`--ai` |

Stage heights shrink downstream (2.4 → 1.0), following the 2D rate labels. The rates move onto the floor ruler, so the stage readouts don't repeat them.

### Sled positions (derived, not hard-coded)
```js
const FEED_GAP = 3.0, HAIDIS_HALF = 2.15;
const sledX = p => rightEdge(PHASES[p].source) + FEED_GAP + HAIDIS_HALF;
// ≈ Phase 1: 8.85   Phase 2: −1.35   Phase 3: −11.25
```
In Phase 2 the sled covers the sunk Add'l Analysis + AI-Ready footprints. In Phase 3 it covers the sunk Cal & Reco + Intermediate footprints, which is the reason those stages have to sink.

### Connectors
- Short `--muted` tubes with a small cone arrowhead between adjacent stages (DAQ→Raw … Add'l→AI).
- Each tube carries a scrolling `FLOW_TEX`. Flow speed and texture density scale with stage data rate: fast and dense near Raw, slow and sparse near AI-Ready.
- When a connector is bypassed, its flow stops and its opacity drops to ~0.08.

### Feed lanes (source → HAIDIS)
- N thin tubes with phase-colored `FLOW_TEX`, one per stream. Lane z-offsets: Phase 1 `[0]`, Phase 2 `[−.3, .3]`, Phase 3 `[−.45, 0, .45]`.
- Leg A curve: source right face (y .6) → HAIDIS notch (low `arc()` lift .4).
- Leg B curve: HAIDIS tip → SAGIPS left face, defined in sled-local space so it moves with the sled.
- Lanes are rebuilt (dispose old geometry) during each transition, while faded out.

---

## HTML readouts

| Anchor | Content | Align |
|---|---|---|
| DAQ | **DAQ / DETECTOR** | above |
| Raw | **RAW DETECTOR / DATA** · unfiltered · status | above |
| Cal & Reco | **CALIBRATION & / RECONSTRUCTION** · status | above |
| Intermediate | **INTERMEDIATE / DATA PRODUCTS** · status | above |
| Add'l Analysis | **ADDITIONAL / ANALYSIS** · status | above |
| AI-Ready | **AI-READY / TRAINING DATA** · curated & labelled · status | above |
| HAIDIS | **HAIDIS TRAINING** · SAGIPS model update · `SOURCE <stage>` · `STREAMS n` · `PKTS n` | above |
| SAGIPS | **SAGIPS MODEL** · trained weights · v1.0 → … · `UPDATES n` | right |
| Ruler ticks | rate strings | below |
| Plot | title, 5 x-tick labels, axis title `M(γ₁,γ₂) [GeV/c²]`, annotations `π⁰ → γγ`, `η → γγ` | per element |

- Two-line headers keep each readout narrower than the ~5-unit stage pitch at Overview zoom.
- Status line: `● SOURCE` in the phase color while a stage feeds HAIDIS; empty while it's simply upstream. When a stage is bypassed, the label switches to `.lbl.ghost` (one line, 45 % opacity, `BYPASSED`) and is re-anchored to the footprint front edge with `below` alignment.
- **Generalize `addLabel` anchors** to `V3 | Object3D | () => V3`. Sled children and plot elements resolve via `getWorldPosition()`. Add `center` alignment and a borderless `.ann` class for plot text.

---

## HUD

- Eyebrow `HAIDIS · Jefferson Lab`; h1 `Adaptive Training Data Pipeline`; sub `HAIDIS trains on any stage of the data processing pipeline — from curated AI-ready data to raw detector output`.
- View buttons: **Overview · Detector · Pipeline · Training** | **Pause**. `Training` follows the sled (see Camera).
- **Phase caption** bottom-center: large display font, colored by phase token, 0.4 s CSS fade on change. Copy the 2D label text.
- **Phase stepper** next to the caption: three chips (`1 AI-Ready`, `2 Intermediate`, `3 Raw`). The active chip is filled with the phase color, and clicking a chip jumps to that phase.
- Legend (2 columns, SVG icons in the training-3D style): Phase 1 / 2 / 3 swatches · Training data feed (flow line) · Data packet · Pipeline flow (line + arrow) · Bypassed stage (dashed footprint).

---

## Choreography (same timings as 2D, collected in one `T` object)

```js
const T = { start:5, phaseRun:6, trainEvery:2.2, trainPkts:5, pktGap:.18, bunchPeriod:2.2,
  flickers:4, flickerGap:.22, plotIn:.7, plotHold:2.8, plotOut:.6,
  labelOut:.4, glowOut:.9, sink:1.0, sinkStagger:.15, slide:1.2, slideDelay:.4,
  feedDelay:.5, feedOut:.3, feedIn:.5, nextPhase:2.2, periodicPlot:7 };
const PHASES = [
  {source:'ai',  token:'--ai',  lanes:[0],           bypass:[],                       label:'Phase 1 — Training on AI-Ready Data'},
  {source:'int', token:'--int', lanes:[-.3,.3],      bypass:['ana','ai'],             label:'Phase 2 — Training on Intermediate Data Products'},
  {source:'raw', token:'--raw', lanes:[-.45,0,.45],  bypass:['cal','int','ana','ai'], label:'Phase 3 — Training directly on Raw Detector Data', trigger:'collision'},
];
```
`bypass` lists are cumulative, so `setPhaseState(i)` is idempotent and jump-to-phase can use it directly.

### Sequence (matches the 2D boot chain)
1. **Intro** (`T.start`): ambient connector flow runs. The DAQ bunch starts orbiting at `T.start`.
2. **startPhase(i)**:
   - Source stage glow sprite and emissive ramp up over 0.7 s.
   - Caption fades in; stepper chip activates.
   - Trains start: timer-driven every 2.2 s in phases 1–2, collision-driven in phase 3.
3. **sendTrain**: 5 packets per lane, 0.18 s apart, colored by phase.
   - Leg A ends: HAIDIS flashes (edge color → white lerp, halo, `PKTS++`).
   - Leg B ends: SAGIPS core pulses (scale + glow, `UPDATES++`).
4. **End of phase 1 / 2** (after `T.phaseRun`): stop spawning → `showPlot()`:
   - SAGIPS core flickers 4×.
   - Panel fades in, bars grow left→right with ~12 ms stagger, HTML plot labels fade in.
   - Hold 2.8 s, then shrink and fade out 0.6 s → transition.
5. **transition(i → i+1)**:
   - Caption fades out; old source glow dims (0.9 s).
   - Newly bypassed stages sink below the floor (`y → −(h+.2)`, 1.0 s, staggered). Dashed footprint `LineLoop`s fade in, readouts go ghost, connector flow stops.
   - Sled slides to `sledX(i+1)` (1.2 s quadInOut, 0.4 s delay). Hover-pad glow brightens while it moves, with a short motion streak.
   - Feed lanes fade out (0.3 s) → rebuild for the new source and lane count → fade in (0.5 s).
   - `startPhase(i+1)` at +2.2 s, then `showTheoryUpdate()`.
6. **showTheoryUpdate**: chevron fades in 0.45 s, then flickers on the 2D keyframes (`.2/.09 1/.09 .15/.07 1/.11 .4/.08 1/.10`), holds 1.3 s, and fades out 0.65 s.
7. **Phase 3** loops forever: each DAQ collision sends a red spark DAQ→Raw, and its arrival launches a train from Raw. `showPlot()` repeats every 7 s after the previous plot ends.

### DAQ collision mechanics
- The bunch angle comes from `simT` (no mover), like the training-3D ring diamonds.
- A collision fires when the bunch crosses the +x ("East") detector, which faces Raw.
- On each collision: the East module flashes (glow sprite), short radial spark lines at the interaction point flash and fade, and a spark travels the DAQ→Raw connector in every phase.

---

## Code architecture (section order inside the `<script>`)

1. Tokens → `COL`, `reduceMotion`, URL params (`?phase=1|2|3`, `?nodelay`, `?view=training`).
2. Renderer / scene / lights / floor (copied from training-3D, shadow camera re-fit to x −28…22).
3. Helpers, copied plus new ones: `extrudeFloorShape(pts, depth)`, `footprintLoop(pts, color)`, `NOISE_TEX`, `textTexture(lines, …)`.
4. Layout constants: `K`, `STAGES[]` (`id, name, x, w, h, kind, token, rate`), `FEED_GAP`, `PHASES[]`, `T`.
5. Builders: `buildDAQ()`, `buildStage(s)` (switch on `kind`: box / chevron / stack / net), `buildConnector(a, b)`, `buildRuler()`, `buildSled()` (HAIDIS + link + SAGIPS + plot + theory as children), `buildPlot()`, `buildTheory()`.
6. Labels with generalized anchors.
7. Sim core:
   - `after(d, fn, tag)` and `cancel(tag)`.
   - `move()` (copied).
   - New `anim({dur, ease, update(u), done, tag})` list for property tweens: sinks, slides, fades.
   - New `keys(seq, apply)` for flicker sequences.
   - Everything advances inside `step(dt)`, so Pause freezes the whole choreography.
8. Feed: packet pool (copied), `rebuildLanes(phase)`, `sendTrain()`, `flashHAIDIS()`, `pulseSAGIPS()`.
9. Choreography: `setPhaseState(i)` (snap: stage sunk/raised, sled x, lanes, glows, labels, caption), `startPhase`, `showPlot`, `transition`, `showTheoryUpdate`, `schedulePeriodicPlot`, `boot()`.
10. `jumpTo(i)`: `cancel('chor')`, clear movers/anims tagged `chor` and release packets, `setPhaseState(i)`, `startPhase(i)`.
11. Camera presets plus sled follow.
12. HUD wiring (views, pause, stepper, legend).
13. Frame loop: `step` / `decay` / billboards (full and yaw-only) / render / `placeLabels`.

### Camera presets (starting values; tune on screen)
| View | Landscape target / θ / φ / r | Notes |
|---|---|---|
| Overview | (−4, .5, 2.5) / 0° / 50° / ≈40 | Fixed. Phase 3 deliberately leaves the sunk footprints in view |
| Detector | (−20.5, 1, 0) / −15° / 52° / 16 | DAQ + Raw close-up |
| Pipeline | (−7.5, 1, 0) / −8° / 50° / 26 | Five processing stages |
| Training | (sled.x + 2.5, 2, 0) / 18° / 45° / 20 | Target follows the sled while active and `!userMoved`, and includes the plot |

Portrait variants use θ = 90° for Overview (the long axis runs vertically on screen) and r multipliers like `presetFor()` in training-3D.

---

## Implementation phases

Each phase leaves a file that opens in a browser by double-click and can be inspected.

### Phase 1 — Scaffold
Copy the training-3D infrastructure: CSS/HUD shell, renderer, floor, helpers, generalized label system, sim core (with `anim`/`keys`/tags), camera presets and orbit, Pause, clock. Empty stage except floor, grid, and the pipeline bed strip.
**Inspect:** orbit, zoom, preset tweens, Pause, resize, portrait layout.

### Phase 2 — Static scene
Build the DAQ ring, five stage objects, connectors, data-rate ruler, sled (HAIDIS + link + SAGIPS) at the Phase 1 position, and all readouts and legend. Build the plot panel and theory chevron hidden.
**Inspect:** silhouettes vs. the 2D shapes, label overlap at 1440×810 and < 720 px, Overview framing, shadow coverage.

### Phase 3 — Ambient motion
- DAQ bunch orbit, collision flash/sparks, DAQ→Raw spark.
- Connector flow with rate-scaled speed.
- Idle details: Raw bit-noise scroll, chevron scan line, Intermediate discs rotating, AI-Ready net nodes pulsing input→hidden→output, SAGIPS core spin.

**Inspect:** motion stays subtle enough that feed packets will still dominate visually.

### Phase 4 — Phase 1 feed loop
Feed lanes, packet trains on Leg A/B, HAIDIS flash, SAGIPS pulse, readout counters, source glow on AI-Ready.
**Inspect:** packet speed (`SPEED = 9`, same as training-3D), flash decay, that packets entering the opaque HAIDIS body read as "consumed".

### Phase 5 — Choreography
`setPhaseState`, `showPlot` (3D bars + HTML plot labels), `transition` (sink, footprints, ghost labels, sled slide, lane rebuild), `showTheoryUpdate`, Phase 3 collision-driven trains, periodic plot, phase caption and stepper display.
**Inspect:** full 2D-equivalent run from t = 0 through the second Phase 3 plot; no in-flight packets orphaned by transitions; histogram matches 2D bins (copy the 40 heights verbatim from `pipeline-overview.html` lines 338–377).

### Phase 6 — Polish and integration
- Stepper jump-to-phase plus `?phase=`/`?nodelay`/`?view=` params.
- Training-view sled follow.
- Reduced-motion path.
- Geometry/material disposal for rebuilt lanes and pulses.
- Pause-safe CSS flashes (use `decay`, not `setTimeout`, where possible).
- **`index.html`**: add a card for the 3D pipeline with a dark-themed static SVG thumbnail (320×160) and a `3D` tag.
- **`CLAUDE.md`**: add the file to the repo layout and add a short "3D animation conventions" section (Three.js r128, dark tokens, sim clock instead of GSAP, label anchoring, camera presets), so later 3D variants stay consistent.

**Inspect:** a screen recording at 1440×810 holds up end-to-end; jumping between phases mid-transition leaves a clean state.

---

## Verification

1. Open `animations/haidis-pipeline-overview-3d.html` directly from disk with no console errors (Three.js r128 + Google Fonts from CDN).
2. After 5 s, Phase 1 green packets flow AI-Ready → HAIDIS → SAGIPS. After 6 s: SAGIPS flickers 4×, the histogram grows (π⁰ peak near 0.135, η near 0.55 GeV/c²), and it fades out.
3. Transition 1→2: Add'l Analysis + AI-Ready sink and leave footprints, the sled glides upstream, gold 2-lane feed from Intermediate, Theory Update flickers.
4. Transition 2→3: Cal & Reco + Intermediate sink, the sled docks next to Raw, and red 3-lane trains fire on each DAQ collision. The plot repeats every ~7 s.
5. Pause freezes everything, including transitions and the plot. Play resumes cleanly.
6. `?phase=3` and the stepper chips land in a correct static state. All four camera views frame their subjects in landscape and portrait.
7. Optional: headless Playwright screenshots at 1440×810 and 390×844 for `?phase=1|2|3`.

---

## Git

Work continues on the existing `feature/3d-animations` branch, which already holds the untracked `haidis-training-phase-3d.html`. Commit and push only after you confirm the changes are complete (per CLAUDE.md).

---

## Open decisions (defaults assumed above)

1. **Bypassed stages:** sink into the floor and leave dashed footprints (default), or stay in place as translucent ghosts like 2D. Ghosts would collide with the sled in Phases 2–3.
2. **Inference plot:** 3D bars on a holographic panel with HTML text (default), or drop the 2D SVG plot verbatim into an HTML panel anchored at SAGIPS (simpler, less 3D).
3. **Gallery:** add a card only for the new 3D pipeline (default), or also add one for `haidis-training-phase-3d.html`.
4. **Phase stepper / jump controls:** included by default because they help when presenting.

---

## Status (2026-11-09)

Phases 1–6 implemented in a single pass in `animations/haidis-pipeline-overview-3d.html`; gallery card added to `index.html`; 3D conventions + repo layout added to `CLAUDE.md`. Open decisions resolved with the defaults above.

Verified headlessly (no browser): inline script parses; choreography state machine (mock THREE) runs the full 2D-equivalent sequence — phase 1 at t=5 s, plot at ~11 s, transition 1→2 at ~16 s (Add'l + AI-ready sink, sled 8.85 → −1.35), phase 2 at ~18 s, transition 2→3 at ~29.5 s (Cal + Int sink, sled → −11.25), phase 3 collision-driven trains + plot every ~7 s; phase jumps mid-transition land in a clean state; camera framing computed for 1440×810, 1920×1080, 1280×800, 390×844, 768×1024.

**Not yet verified:** actual WebGL rendering — label overlap, colours, canvas-texture text, shadow coverage need an in-browser look.

Deviations from the plan: lanes re-route once the sled has docked (not mid-slide); stages sink downstream-first; added `?seek=<s>` for screenshots.
