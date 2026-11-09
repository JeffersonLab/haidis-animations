# HAIDIS Animations — CLAUDE.md

## Project purpose
Static HTML animations for HAIDIS project presentations, demos, and meetings — 2D (SVG + GSAP) and 3D (Three.js) variants. Designed to be screen-recorded at 16:9 (1440×810) or opened directly in a browser. No build step, no server required — open `index.html` directly in a browser.

## Repository layout
```
index.html                      ← Gallery/selector page (one card per animation)
animations/
  training-phase.html           ← Training topology: data senders → LB → workers → MLFlow
  pipeline-overview.html        ← Adaptive pipeline: 3-phase training data progression
  node-detail.html              ← Perlmutter node internals: ERSAP → SHMem → SAGIPS ranks · MPI ring · MLFlow
  haidis-training-phase-3d.html ← Three.js variant of training-phase (dark theme, orbit camera)
  haidis-pipeline-overview-3d.html ← Three.js variant of pipeline-overview (sled moves upstream, stages sink)
plans/
  node-detail.md                ← Implementation plan for node-detail (phased)
  pipeline-overview-3d.md       ← Implementation plan for haidis-pipeline-overview-3d
README.md
CLAUDE.md
```

## Tech stack
- **2D animations**: **SVG** for all layout and visual elements (viewBox 1440×810, 16:9) + **GSAP 3** (cdnjs) for JavaScript-driven animation
- **3D animations** (`*-3d.html`): **Three.js r128** (cdnjs), no GSAP — see "3D animation conventions" below
- **Pure CSS** for the gallery page styling and card hover effects
- **No build tool, no npm, no framework** — everything is a single self-contained HTML file

## Adding a new animation

1. Create `animations/<phase-name>.html` following the conventions below (2D) or `animations/haidis-<phase-name>-3d.html` (3D).
2. In `index.html`, add an `<a class="card" href="animations/<file>.html">` card to the grid (2D cards first, then 3D cards; use a `class="card placeholder"` block for planned-but-unbuilt animations).
3. Add a miniature SVG thumbnail inside `.card-thumb` (viewBox 320×160, same topology layout, no GSAP). 3D cards use `class="card-thumb thumb-3d"`.

## Animation file conventions (2D: SVG + GSAP)

### Canvas
```html
<svg id="scene" viewBox="0 0 1440 810" xmlns="http://www.w3.org/2000/svg">
```
CSS keeps it letterboxed at 16:9:
```css
svg { width: 100vw; height: 100vh; max-width: 177.78vh; max-height: 56.25vw; }
```
If the scene needs more vertical space (e.g. a legend that would clip below 810px), expand the viewBox height — `node-detail.html` uses `1440×845`. The letterbox CSS still scales correctly; the scene just renders slightly taller than a pure 16:9 frame at the same width.

### Background
- Body bg: `#e8eef6`
- SVG bg: radial gradient from `#f2f6fc` (center) to `#e4eaf4` (edge)
- Subtle grid: `stroke="#a8bdd4"`, `stroke-width="0.5"`, `opacity="0.3"`, 160px cell spacing

### Color palette (node types)
| Component | Stroke / accent | Text |
|---|---|---|
| GlueX Data Sender 1 | `#0099cc` (cyan) | `#006890` |
| GlueX Data Sender 2 | `#e05820` (orange) | `#b03808` |
| EJFAT Load Balancer | `#c89000` (gold) | `#9a7000` |
| ERSAP+SAGIPS Workers | `#4a70a0` (blue) | `#2a4870` |
| AmSC MLFlow Registry | `#1a9060` (green) | `#1a5040` |
| Ring (model weights) | `#9933bb` (purple) | `#7722aa` |
| Data packets S1 | `#0099cc` | — |
| Data packets S2 | `#e05820` | — |
| Raw Detector Data | `#cc3344` (red) | `#aa2233` |
| Calibration & Reco | `#e05820` (orange) | `#b03808` |
| Intermediate Data | `#c89000` (gold) | `#9a7000` |
| Additional Analysis | `#4a70a0` (blue) | `#2a4870` |
| AI-Ready Data | `#1a9060` (green) | `#1a5040` |
| HAIDIS Training | `#9933bb` (purple) | `#7722aa` |
| SAGIPS Model | `#0099cc` (cyan) | `#006890` |

### Node style
- White fill (`#ffffff`), colored stroke 1.5–2px, `rx="6–10"`, drop-shadow filter
- 5px colored left-accent bar inside each node box
- Node boxes are tall enough to hold 3–4 lines of text comfortably (~80–110px height)
- Font: `'Courier New', monospace` throughout
- `filter="url(#shadow-node)"` on every node `<g>`

### Node shapes
- **Rectangle** (`<rect>`): standard boxes — sender nodes, data product boxes, SAGIPS
- **Chevron** (`<polygon>`): processing step nodes — `"x0,top x1,top x1+18,cy x1,bottom x0,bottom"` with a flat left accent bar
- **HAIDIS chevron** has a left-side V-notch: `"900,350 1090,350 1115,405 1090,460 900,460 920,405"` with matching notch accent polygon

### SVG filters (define in `<defs>`)
```
glow-cyan, glow-orange, glow-purple, glow-gold, glow-green  ← feDropShadow glows
shadow-node   ← subtle drop shadow for node boxes
blur-glow     ← feGaussianBlur stdDeviation=11 (for active-source glow rects)
```

### Glow / active-source highlight (pipeline-overview pattern)
Each source node has a blurred `<rect>` sibling drawn **before** the node (behind it in z-order):
```svg
<rect id="glow-raw" x="126" y="349" width="118" height="112" rx="10"
      fill="#cc3344" opacity="0" filter="url(#blur-glow)"/>
```
Activate: `gsap.to('#glow-raw', { opacity: 0.35, duration: 0.7 })`

### Links
- Solid `stroke="#8aaac8"` 2px → data forward paths
- Dashed `stroke="#a8c0d8"` 1.5px `stroke-dasharray="6,5"` → ready/feedback signals
- Dashed `stroke="#9933bb"` 1.8px → ring arcs between workers
- Dashed `stroke="#1ab870"` 1.3px → worker → MLFlow save paths
- Dashed colored line (phase-color, stroke-width=3.5) → training feed line in pipeline-overview

### Animated data packets
- Created dynamically via JS (`document.createElementNS`)
- 14×9px rect with a soft halo rect behind it
- **training-phase**: moved with `gsap.to(el, { attr: { x, y }, duration, ease:'none' })`
- **pipeline-overview**: parent `<g>` translated with `gsap.to(g, { x: dx })` (CSS transform)
- Speed constant `TRAVEL_SPEED = 280` (SVG units/sec)
- Packets disappear when their **right edge** reaches the **left border** of the destination node
- Appended to a dedicated `<g id="packets">` layer (always on top of nodes in SVG order)

### GSAP patterns used
```js
gsap.delayedCall(delay, callback)          // schedule a burst
gsap.to(el, { attr: {...}, duration, ease:'none', onComplete })
gsap.to(el, { x: dx, duration, ease:'power2.inOut' })  // CSS transform slide
gsap.timeline()                             // for sequenced phase animations
gsap.fromTo(el, { opacity:0 }, { opacity:1 })
```

### Frame timer (live clock)
Use a `requestAnimationFrame` IIFE — do NOT use GSAP for this:
```js
(function () {
  const counter = document.getElementById('frame-counter');
  const t0 = performance.now();
  function tick() {
    counter.textContent = 't = ' + ((performance.now() - t0) / 1000).toFixed(1) + ' s';
    requestAnimationFrame(tick);
  }
  requestAnimationFrame(tick);
})();
```

### `flashEl` re-entrancy pitfall
When a node box is flashed on every packet arrival and multiple packets can arrive before the flash decays, naively reading `stroke-width` mid-tween will ratchet the border inward on each call. Fix: cache the baseline once in `dataset`, kill in-flight tweens, then reset before flashing:
```js
function flashEl(groupId, brightColor, duration) {
  const grp = document.getElementById(groupId);
  const r   = grp.querySelector('rect');
  if (!r.dataset.baseStroke) {
    r.dataset.baseStroke = r.getAttribute('stroke');
    r.dataset.baseWidth  = r.getAttribute('stroke-width') || '1.5';
  }
  const origStroke = r.dataset.baseStroke;
  const origWidth  = parseFloat(r.dataset.baseWidth);
  gsap.killTweensOf(r);
  gsap.set(r, { attr: { stroke: origStroke, 'stroke-width': origWidth } });
  gsap.to(r, {
    attr: { stroke: brightColor, 'stroke-width': origWidth + 1.5 }, duration: 0.1,
    onComplete: () => gsap.to(r, {
      attr: { stroke: origStroke, 'stroke-width': origWidth }, duration: duration || 0.3
    })
  });
}
```

### Feed line transition (pipeline-overview)
Do NOT use `gsap.to(line, { attr: { x1: newX } })` — SVG presentation attributes do not interpolate. Use a timeline:
```js
const tl = gsap.timeline();
tl.to(feedLine, { opacity: 0, duration: 0.3 });
tl.call(() => {
  feedLine.setAttribute('x1', newX1);
  feedLine.setAttribute('stroke', newColor);
});
tl.to(feedLine, { opacity: 0.9, duration: 0.5 });
```

### Text / labels
- Title bar: **18px**, `letter-spacing:3`, `fill="#4a6080"`, centered at top
- Subtitle: **14px**, `letter-spacing:1.5`, `fill="#6a80a0"`, centered below title
- Node headers: **13px** bold, `letter-spacing:1`
- Sub-labels: **10–11px**
- Micro stats/counters: **9–10px**
- Phase label: **17px** bold, centered at bottom of scene
- Frame timer: bottom-right, `fill="#8090a8"`, `font-size:9`

### Legend
Bottom-left box showing all link types and packet colors. Use the same visual primitives (small rects, polylines, dashed lines, polygon diamonds) as appear in the scene.
- Font: **11px**, row pitch: **26px** for single-column; **18px** for two-column dense layout
- Box sized to contain all rows with 8px top/bottom padding and 6px left/right margin
- All icons centered vertically on their text row (icon center-y = text baseline - ~4px)
- For dense legends (8+ entries), use a 2-column layout: two icon+label groups per row, each column occupying half the box width

## 3D animation conventions (`*-3d.html`)

The 3D variants deliberately differ from the SVG conventions above. Copy the scaffolding from an existing 3D file rather than starting fresh.

- **Stack**: Three.js **r128** from cdnjs (`three.min.js`), single self-contained file, **no GSAP**. Full-bleed `<canvas id="scene">` (not letterboxed); HUD is HTML overlay.
- **Theme**: dark instrument-room stage, tokens in `:root` and read into `COL` via `getComputedStyle` (`--bg #0c1524`, `--panel`, `--line`, `--fg`, `--muted`). Brightened palette for emissive readability: S1/SAGIPS `#22b8e8`, S2/Cal `#ff7a3d`, LB/Intermediate `#e2b13c`, workers/Add'l analysis `#7ea2d6`, MLFlow/AI-ready `#2fd48a`, ring/HAIDIS `#b65ae6`, raw `#ee4b5c`.
- **Fonts**: Barlow Semi Condensed (display) + IBM Plex Mono (readouts) from Google Fonts. Canvas-texture text must redraw after `document.fonts.load(...)` resolves.
- **Readouts**: HTML `.lbl` panels anchored to 3D points and re-projected every frame (`addLabel(html, anchor, cls, align, cssVar)` + `placeLabels()`). Anchors may be a `Vector3` or an `Object3D` (follows moving groups). `.sec` lines hide below 720 px.
- **Time**: a pausable sim clock (`simT`) drives everything — `after(delay, fn, tag)` scheduler, `move()` curve movers, `anim()/tw()/keys()` value tweens. `step(dt)` advances logic, `decay(dt)` eases flashes, `pose()` sets idle motion from `simT`. Pause = stop calling `step`. Tag choreography (`'chor'`) so `cancelTag` can reset it (phase jumps).
- **Camera**: named presets with landscape/portrait variants, drag-orbit + wheel/pinch zoom, 1.2 s tween between presets; `userMoved` suppresses auto-reframing on resize. View buttons + Pause in the top-right HUD.
- **Shapes**: reuse 2D SVG polygons by scaling (`K = 1/50`) and extruding (`svgShape` → `ExtrudeGeometry`, rotated so SVG y → world +z).
- **Accessibility**: `prefers-reduced-motion` starts paused with instant camera moves; canvas has a descriptive `aria-label`.
- **URL params** (pipeline-overview-3d): `?phase=1|2|3`, `?nodelay`, `?view=overview|detector|pipeline|training`, `?seek=<s>`.
- **Gallery thumbnail**: `.card-thumb.thumb-3d` (dark `#0c1524`), pseudo-isometric blocks (front rect + lighter top parallelogram + darker side face) in the 3D palette, `card-tag` "3D · Three.js · interactive", "3D <NAME>" corner tag at top-left. Prefix SVG `id`s per card (e.g. `tp3d-`, `pov3d-`) — all thumbnails share one document.
- **Verifying without a browser**: `node --check` on the extracted inline `<script>` catches syntax errors; `rsvg-convert` renders gallery thumbnails to PNG (static frame only — SMIL motion is not evaluated).

## HAIDIS system components reference
| Element | Real system |
|---|---|
| GlueX Data 1/2 | Physics data senders, 10.0.1.10–11:19522, UDP 10 Gb/s, 2048-byte frames |
| Load Balancer | EJFAT FPGA, round-robin + backpressure, feedback from workers |
| Workers 1–5 | ERSAP + SAGIPS nodes, 10.0.2.10–14:19530 |
| Ring topology | MPI rank communications, shared model weights |
| AmSC MLFlow | Model snapshot registry, versioned saves |
| Pipeline stages | Raw Detector → Cal & Reco → Intermediate Data → Additional Analysis → AI-Ready |
| HAIDIS insight | Can train directly on any stage — eliminates upstream processing steps |

## Git workflow

1. **New change request → new feature branch.** For every new change request, create a feature branch from `main` before making any edits:
   ```bash
   git checkout main && git pull
   git checkout -b feature/<short-description>
   ```
2. **Confirm completion before committing.** After all edits are done, check with the user that the changes are complete. Once confirmed, stage and commit the relevant files, then push the branch to origin:
   ```bash
   git push -u origin feature/<short-description>
   ```
3. **Returning to main always pulls from origin.** Whenever switching back to `main`, always follow with a pull:
   ```bash
   git checkout main && git pull
   ```
4. **Never commit directly to main.** All changes must go through a feature branch and be pushed to origin from that branch.

## Gallery page (index.html) conventions
- Grid of `.card` links, `minmax(320px, 1fr)`, max-width 1100px
- Each card: `.card-thumb` (160px, static SVG preview) + `.card-body` (tag / title / desc / footer)
- Badge colors: LIVE = green (`#e8f4ee` / `#1a6040`), PENDING = grey
- Placeholder cards: `class="card placeholder"` with `opacity:0.5; pointer-events:none`
- Thumbnail SVG: viewBox `0 0 320 160`, same color palette, no GSAP. SMIL declarative animation (`<animateTransform>`, `<animate>`) is fine for packet motion and ring diamonds — use the opacity trick to hide the return trip: `values="0.85;0.85;0;0;0.85"` with matching `keyTimes`
- For curved paths, `<animateMotion path="…">` (or `<mpath href="#id"/>` to reuse a drawn path) is simpler than translate keyframes; `keyPoints="0;1;1" keyTimes="0;0.4;1" calcMode="linear"` plus a matching opacity animation gives a travel-then-hide cycle
- 3D animation cards follow the 2D cards and use the dark `.thumb-3d` thumbnail style (see 3D conventions)
