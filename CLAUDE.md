# HAIDIS Animations — CLAUDE.md

## Project purpose
Static HTML+SVG animations for HAIDIS project presentations, demos, and meetings. Designed to be screen-recorded at 16:9 (1440×810) or opened directly in a browser. No build step, no server required — open `index.html` directly in a browser.

## Repository layout
```
index.html                      ← Gallery/selector page (one card per animation)
animations/
  training-phase.html           ← Training topology: data senders → LB → workers → MLFlow
  pipeline-overview.html        ← Adaptive pipeline: 3-phase training data progression
README.md
CLAUDE.md
```

## Tech stack
- **SVG** for all layout and visual elements (viewBox 1440×810, 16:9)
- **GSAP 3** (loaded from cdnjs CDN) for all JavaScript-driven animation
- **Pure CSS** for the gallery page styling and card hover effects
- **No build tool, no npm, no framework** — everything is a single self-contained HTML file

## Adding a new animation

1. Create `animations/<phase-name>.html` following the conventions below.
2. In `index.html`, replace the next `<div class="card placeholder">` block with a real `<a class="card" href="animations/<phase-name>.html">` card.
3. Add a miniature static SVG thumbnail inside `.card-thumb` (viewBox 320×160, same topology layout, no GSAP).

## Animation file conventions

### Canvas
```html
<svg id="scene" viewBox="0 0 1440 810" xmlns="http://www.w3.org/2000/svg">
```
CSS keeps it letterboxed at 16:9:
```css
svg { width: 100vw; height: 100vh; max-width: 177.78vh; max-height: 56.25vw; }
```

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
- Font: **11px**, row pitch: **26px**
- Box sized to contain all rows with 8px top/bottom padding and 6px left/right margin
- All icons centered vertically on their text row (icon center-y = text baseline - ~4px)

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

## Gallery page (index.html) conventions
- Grid of `.card` links, `minmax(320px, 1fr)`, max-width 1100px
- Each card: `.card-thumb` (160px, static SVG preview) + `.card-body` (tag / title / desc / footer)
- Badge colors: LIVE = green (`#e8f4ee` / `#1a6040`), PENDING = grey
- Placeholder cards: `class="card placeholder"` with `opacity:0.5; pointer-events:none`
- Thumbnail SVG: viewBox `0 0 320 160`, same color palette, no animation
