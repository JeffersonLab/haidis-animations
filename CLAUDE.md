# HAIDIS Animations — CLAUDE.md

## Project purpose
Static HTML+SVG animations for HAIDIS project presentations, demos, and meetings. Designed to be screen-recorded at 16:9 (1440×810) or opened directly in a browser. No build step, no server required — open `index.html` directly in a browser.

## Repository layout
```
index.html                  ← Gallery/selector page (one card per animation)
animations/
  training-phase.html       ← Phase 1: data ingestion & training topology
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

### Node style
- White fill (`#ffffff`), colored stroke 1.5–2px, `rx="6–10"`, drop-shadow filter
- 5px colored left-accent bar inside each node box
- Font: `'Courier New', monospace` throughout
- `filter="url(#shadow-node)"` on every node `<g>`

### SVG filters (define in `<defs>`)
```
glow-cyan, glow-orange, glow-purple, glow-gold, glow-green  ← feDropShadow glows
shadow-node  ← subtle drop shadow for node boxes
```

### Links
- Solid `stroke="#8aaac8"` 2px → data forward paths
- Dashed `stroke="#a8c0d8"` 1.5px `stroke-dasharray="6,5"` → ready/feedback signals
- Dashed `stroke="#9933bb"` 1.8px → ring arcs between workers
- Dashed `stroke="#1ab870"` 1.3px → worker → MLFlow save paths

### Animated data packets
- Created dynamically via JS (`document.createElementNS`)
- 14×9px rect with a soft halo rect behind it
- Moved with `gsap.to(el, { attr: { x, y }, duration, ease:'none' })`
- Speed constant `TRAVEL_SPEED = 280` (SVG units/sec)
- Appended to a dedicated `<g id="packets">` layer (always on top of nodes in SVG order)

### GSAP patterns used
```js
gsap.delayedCall(delay, callback)          // schedule a burst
gsap.to(el, { attr: {...}, duration, ease:'none', onComplete })
gsap.timeline()                             // for sequenced phase animations
```

### Text / labels
- Title bar: 13px, `letter-spacing:3`, `fill="#4a6080"`, centered at top
- Node headers: 11px bold, `letter-spacing:1`
- Sub-labels: 8.5–9.5px
- Micro stats/counters: 7–9px
- Frame timer: bottom-right, `fill="#8090a8"`, `font-size:9`

### Legend
Bottom-left box showing all link types and packet colors. Use the same visual primitives (small rects, polylines, dashed lines, polygon diamonds) as appear in the scene.

## HAIDIS system components reference
| Element | Real system |
|---|---|
| GlueX Data 1/2 | Physics data senders, 10.0.1.10–11:19522, UDP 10 Gb/s, 2048-byte frames |
| Load Balancer | EJFAT FPGA, round-robin + backpressure, feedback from workers |
| Workers 1–5 | ERSAP + SAGIPS nodes, 10.0.2.10–14:19530 |
| Ring topology | MPI rank communications, shared model weights |
| AmSC MLFlow | Model snapshot registry, versioned saves |

## Gallery page (index.html) conventions
- Grid of `.card` links, `minmax(320px, 1fr)`, max-width 1100px
- Each card: `.card-thumb` (160px, static SVG preview) + `.card-body` (tag / title / desc / footer)
- Badge colors: LIVE = green (`#e8f4ee` / `#1a6040`), PENDING = grey
- Placeholder cards: `class="card placeholder"` with `opacity:0.5; pointer-events:none`
- Thumbnail SVG: viewBox `0 0 320 160`, same color palette, no animation
