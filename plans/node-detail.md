# Plan: `animations/node-detail.html` — Perlmutter Node Detail Animation

## Context

The existing repo has two animations (training-phase, pipeline-overview) that show the HAIDIS system at a macro level. This new animation zooms in to show **what happens inside a single Perlmutter compute node**: the ERSAP container (E2SAR Receiver → Event Queue → GlueX Analyzer → SHMem Interface) feeding processed events into a shared memory segment that SAGIPS GPU processes consume, while those processes exchange model weights via an intra- and inter-node MPI ring.

Two nodes are stacked vertically so both can be shown simultaneously. The EJFAT LB (data plane + control plane) appears as a compact block on the far left.

No card is added to `index.html` yet.

---

## File to create

`animations/node-detail.html`

---

## SVG Layout (1440×810 viewBox)

### Overall zones

```
  x=12-138     x=153-1425
  ┌──────┐     ┌────────────────────────────────────────────────────────┐
  │EJFAT │────▶│ PERLMUTTER NODE 1  (y=58–383)                         │
  │  LB  │────▶│  [ERSAP container]  [SHMEM]  [SAGIPS container]       │
  │      │     ├────────────────────────────────────────────────────────┤  gap y=383-423
  │      │────▶│ PERLMUTTER NODE 2  (y=423–748)                        │
  │      │────▶│  [ERSAP container]  [SHMEM]  [SAGIPS container]       │
  └──────┘     └────────────────────────────────────────────────────────┘
```

### Coordinates

| Element | x | dy (from node_top) | w | h |
|---|---|---|---|---|
| EJFAT LB block | 12 | — | 126 | 250 (y=280–530) |
| Node box | 153 | 0 | 1272 | 325 |
| ERSAP sub-container | 161 | 6 | 560 | 313 |
| E2SAR Receiver | 170 | 52 | 136 | 218 |
| Event Queue icon | 319 | 96 | 42 | 124 |
| GlueX Event Analyzer | 374 | 52 | 136 | 218 |
| SHMem Interface | 524 | 52 | 136 | 218 |
| SHMEM segment | 732 | 48 | 68 | 245 |
| SAGIPS sub-container | 820 | 6 | 603 | 313 |
| SAGIPS proc row i (i=0–3) | 830 | 52+68*i | 130 | 60 |
| GPU sub-box row i | 975 | 52+68*i | 96 | 60 |
| Intra-node ring arcs | 1079–1160 | — | — | — |

**node1_top = 58**, **node2_top = 423**

All ERSAP block centers share **center_y = node_top + 161** for horizontal arrows.

---

## Static SVG elements

### Filters / defs (reuse from other animations)
- `glow-cyan, glow-orange, glow-purple, glow-gold, glow-green` — feDropShadow
- `shadow-node` — subtle drop shadow
- `blur-glow` (stdDeviation=11) — SHMEM write/read pulse halos
- `bg-grad` — radial gradient (#f2f6fc → #e4eaf4)
- `arrowhead` marker — #8aaac8 filled triangle

### EJFAT LB block (gold, #c89000)
- Box: x=12, y=280, w=126, h=250. White fill, gold stroke 1.8px, rx=8, shadow-node
- Labels: "EJFAT", "LB / FPGA", sub-labels "DATA PLANE" and "CTL PLANE"
- Two labeled connection points for data outputs (y=350 and y=458)
- Data lines (solid #8aaac8, w=1.8): (138,350)→(170,219) for Node 1; (138,458)→(170,584) for Node 2
- Control feedback lines (dashed #a8c0d8, w=1.3, dasharray=5,4): same path but reverse direction, drawn slightly offset

### Perlmutter Node boxes
- Large rounded rect (rx=8), fill=#f4f8ff, stroke=#4a70a0, stroke-width=1.5, shadow-node
- Label "PERLMUTTER NODE 1 / 2" at top-left inside box, fill=#2a4870, 11px bold letter-spacing=2

### ERSAP sub-container
- Dashed rect (stroke-dasharray=6,4), stroke=#4a70a0 opacity=0.6, fill=rgba(240,246,255,0.6)
- Label "ERSAP CONTAINER" centered top, 10px, #4a70a0

### E2SAR Receiver block (cyan #0099cc)
- White fill, cyan stroke 1.5px, rx=6, 5px left accent bar, shadow-node
- Labels: "E2SAR", "RECEIVER", sub: "E2SAR event queue →"
- Status dot (`id="e2sar-dot-N"`) green circle r=4

### Event Queue icon (gold #c89000)
- Narrow rect (42×124), fill=#fffef5, stroke=#c89000, rx=4, shadow-node
- Inside: 5 horizontal bars (like FIFO slots), 20px apart
- Label "EVENT QUEUE" below (9px)
- Depth counter text `id="queue-depth-N"` below label

### GlueX Event Analyzer (orange #e05820)
- Chevron polygon (same pattern as pipeline-overview calreco node, flattened)
  - points="374,node_top+52  510,node_top+52  528,node_top+161  510,node_top+270  374,node_top+270"
- White fill, orange stroke 1.5px, 5px flat left accent
- Labels: "GlueX EVENT", "ANALYZER", sub: "particle ID"

### SHMem Interface (blue #4a70a0)
- Rect, white fill, blue stroke 1.5px, rx=6, 5px left accent, shadow-node
- Labels: "SHMem", "INTERFACE", sub: "writer · semaphore"
- Write counter `id="shm-writes-N"`

### SHMEM segment (purple #9933bb tint)
- Rect x=732, dy=48, w=68, h=245, fill=#f8f0ff, stroke=#9933bb, rx=4, stroke-width=1.5
- Inside: 10 thin horizontal stripes (stroke=#cc88ee, opacity=0.3, every 22px)
- "SHMEM" label centered top (9px bold, #7722aa)
- "SEM" badge (small pill, fill=#9933bb, text white, 8px) below header — indicates semaphore
- Blur-glow rect (same size, id="shmem-glow-write-N" fill=#4a70a0) and another (id="shmem-glow-read-N" fill=#0099cc), both opacity=0, for write/read pulses

### SAGIPS sub-container
- Dashed rect, stroke=#9933bb opacity=0.5, fill=rgba(248,240,255,0.5)
- Label "SAGIPS CONTAINER" centered top, 10px, #7722aa

### SAGIPS process rows (4 per node)
Each row has:
- **Process box** (x=830, w=130, h=60): fill=white, stroke=#4a70a0 1.5px, rx=5, 4px left accent
  - Labels: "PROC i" (11px bold), "rank i" (9px), "training" (9px)
  - id="proc-N-i"
- **GPU sub-box** (x=975, w=96, h=60): fill=#f0f8ff, stroke=#0099cc 1.3px, rx=4
  - Labels: "GPU i" (10px bold), small gear icon (3 arc lines)
  - id="gpu-N-i"
- Arrow between them at center_y (x=960→975, #8aaac8 w=1.5 with arrowhead)

### Intra-node ring arcs (right side of SAGIPS, x=1079–1160)
Four cubic-Bézier arcs connecting consecutive GPU right edges (x=1071 → x=1110 → x=1071):
```
row 0 → row 1: M 1071,center0 C 1148,center0 1148,center1 1071,center1
row 1 → row 2: (same pattern, next row pair)
row 2 → row 3: ...
row 3 → row 0: M 1071,center3 C 1165,center3+40 1165,center0-40 1071,center0  (wrap-around)
```
Color: #9933bb, stroke-width=1.8, opacity=0.6

### Inter-node ring arcs (far right, x=1200–1420)
Two arcs connecting node1's GPU row 3 right edge to node2's GPU row 0 right edge, and vice versa:
```
M 1071,node1_center3 C 1380,node1_center3+60 1380,node2_center0-60 1071,node2_center0
```
Color: #9933bb, stroke-width=2, opacity=0.5, stroke-dasharray=8,5
Label "MPI RING (inter-node)" rotated -90° at x=1400, centered between nodes

### Static connection arrows (drawn before nodes)
- EJFAT data lines (solid #8aaac8, w=1.8): (138,350)→(170,node1_e2sar_y) and (138,458)→(170,node2_e2sar_y)
- Control feedback stubs (dashed #a8c0d8, w=1.2): reverse direction, slightly offset
- ERSAP inter-block arrows at center_y (x=306→319, x=361→374, x=510→524, x=660→732)
- SAGIPS: process→GPU arrows at each row (x=960→975)
- "MODEL WEIGHTS" rotated label between ring arcs

### Legend (bottom-left)
4 rows × 26px pitch, box sized to fit:
- Row 1: cyan packet rect + "Event data" / orange packet + "Processed event"
- Row 2: purple diamond + "Model weights (ring)" / dashed circle + "Control plane signal"
- Row 3: purple SHMEM stripe swatch + "Shared memory write" / cyan swatch + "SAGIPS read"
- Row 4: purple arc + "MPI Rank comms (intra-node)" / curved purple arc + "(inter-node)"

---

## Animation Loops (all factored into named functions)

### Node configuration objects
```js
const NODES = [
  { id: 0, top: 58,  ejfat_line_y: 350 },
  { id: 1, top: 423, ejfat_line_y: 458 },
];
```
All coordinate arithmetic derives from `node.top` — no hard-coded per-node numbers inside loop functions.

### `schedEventArrival(node, firstDelay)`
Per-node event flow loop. Chains via `gsap.delayedCall` + `onComplete`:

1. Spawn packet at EJFAT right edge → fly to E2SAR Receiver left edge (solid, colored #0099cc or #e05820 alternating)
2. **E2SAR flash**: briefly stroke→cyan bright, status dot pulse; increment queue depth display
3. `gsap.delayedCall(0.3–0.6)`: spawn packet from Queue right edge → Analyzer left edge (gold #c89000 tint), decrement queue depth
4. **Analyzer flash**: brief highlight + process delay `0.2–0.4s`
5. `gsap.delayedCall(analyzerDelay)`: spawn packet Analyzer right edge → SHMem Interface left edge
6. **SHMem Interface flash**: brief highlight, increment `shm-writes-N`
7. `gsap.delayedCall(0.15)`: pulse `shmem-glow-write-N` (opacity 0→0.4→0)
8. `gsap.delayedCall(0.25)`: pick random SAGIPS proc i, spawn particle SHMEM right edge → proc i left edge; pulse `shmem-glow-read-N`, flash proc box + GPU box
9. Reschedule: `schedEventArrival(node, interval + randomJitter)`

All sub-steps implemented as inner helper functions: `flyPkt(x1,y1,x2,y2,color,onDone)`, `flashEl(el, brightStroke, duration)`.

### `schedControlFeedback(node, firstDelay)`
Sends small dashed circles from E2SAR Receiver left edge back to EJFAT (same pattern as `schedFeedback` in training-phase.html). Reschedules at `1.0 + random()*0.8s`.

### `startIntraNodeRing(node)`
Identical to `startRing()` in training-phase.html but using the 4 GPU-right-edge points within this node. 3 ring diamonds, `RING_DURATION=4.5s`, cubic-Bézier path across 4 segments. Uses `cubicBez()` helper (copy from training-phase).

### `startInterNodeRing()`
2 diamonds traveling the large inter-node arc (one each direction). Uses the same `gsap.to(state, {t, repeat:-1, onUpdate})` pattern, path is a single large cubic Bézier arc node1→node2→node1.

### Frame timer
`requestAnimationFrame` tick updating `id="frame-counter"`.

---

## Key implementation notes

- **GSAP patterns**: `gsap.delayedCall`, `gsap.to(el, {attr:{...}})`, `gsap.timeline()` where needed — exactly like existing animations.
- **Packet creation**: same `makeDataPkt(cx, cy, color, filter)` + `gsap.to(p.rect, {attr:{x,y}})` as training-phase.html.
- **No hard-coded y per node inside event loops** — all derived from `node.top`.
- **Queue depth counter**: simple JS integer per node, displayed in the Event Queue icon.
- **Random SAGIPS proc selection**: `Math.floor(Math.random() * 4)`.
- **Startup delay**: 5-second `gsap.delayedCall` before first events (consistent with other animations).
- **No GSAP `attr` interpolation for SVG presentation attributes** — use opacity fade+setAttribute for any color changes (per CLAUDE.md warning).

---

## Implementation Phases

Each phase produces a file that can be opened directly in a browser for inspection. All phases build on the same `animations/node-detail.html` — the file is committed (or checkpointed) at the end of each phase.

---

### Phase 1a — Static SVG skeleton: boilerplate, defs, EJFAT, nodes, ERSAP pipeline
**Deliverable**: a partially-complete static diagram showing the left half of each node — everything up to and including the ERSAP pipeline. No animation, no JavaScript.

What to build:
- HTML boilerplate, `<style>` (letterbox CSS), GSAP CDN tag (present but unused)
- All `<defs>`: filters (`glow-*`, `shadow-node`, `blur-glow`), gradients, `arrowhead` marker
- Background rect + grid lines
- Title and subtitle text
- EJFAT LB block with all labels; data-plane lines and control-plane dashed stubs drawn as static `<line>` elements
- Both Perlmutter Node bounding boxes with node labels
- Both ERSAP sub-containers with:
  - E2SAR Receiver block (cyan), Event Queue icon (gold, with 5 slot bars), GlueX Event Analyzer chevron (orange), SHMem Interface block (blue)
  - Static inter-block arrows (EJFAT→E2SAR, E2SAR→Queue, Queue→Analyzer, Analyzer→SHMem Interface, SHMem Interface→SHMEM)
- `<g id="packets">` layer (empty, appended last so it stays on top)

**Inspect**: EJFAT block placement, ERSAP pipeline layout and arrow routing, label readability, color palette.

---

### Phase 1b — Static SVG skeleton: SHMEM, SAGIPS, ring arcs, legend
**Deliverable**: the fully-complete static diagram — adds the right half of each node and all decoration. No animation, no JavaScript.

What to add (continuing the same file):
- Both SHMEM segments (purple, with stripe pattern, "SHMEM" + "SEM" labels; glow rects present but opacity=0)
- Both SAGIPS sub-containers with all 4 process+GPU rows and static proc→GPU arrows
- Intra-node ring arc paths (both nodes) drawn as static `<path>` elements
- Inter-node ring arc paths drawn as static `<path>` elements with "MPI RING (inter-node)" label
- Legend box (bottom-left) with all 4 rows
- Frame timer placeholder text (static "t=0.0s")

**Inspect**: full layout proportions, SHMEM↔SAGIPS spacing, ring arc shapes, legend completeness, no elements clipped outside 1440×810.

---

### Phase 2 — Event data-flow animation
**Deliverable**: packets flow end-to-end through the ERSAP pipeline and into SAGIPS, for both nodes.

What to add (pure JavaScript, no SVG changes):
- `NODES` config array (`id`, `top`, `ejfat_line_y`, per-node counters/state)
- Shared helpers: `dist()`, `dur()`, `TRAVEL_SPEED=280`, `makeDataPkt()`, `flyPkt()`, `flashEl()`
- **`schedEventArrival(node, firstDelay)`** — the full sequential chain:
  1. EJFAT right edge → E2SAR Receiver left edge (packet flies, E2SAR flashes + queue depth++)
  2. Queue right edge → GlueX Analyzer left edge (queue depth--, Analyzer flashes)
  3. Analyzer right edge → SHMem Interface left edge (SHMem Interface flashes, write counter++)
  4. SHMem Interface right edge → SHMEM segment (write pulse on `shmem-glow-write-N`)
  5. SHMEM right edge → random SAGIPS process left edge (read pulse on `shmem-glow-read-N`, process+GPU flash)
  6. Reschedule with jitter
- 5-second startup delay via `gsap.delayedCall`; Node 1 starts at t=5s, Node 2 at t=6.2s

**Inspect**: end-to-end packet flow, flash timing, queue depth counter incrementing/decrementing, SHMEM write/read pulses, random SAGIPS process selection.

---

### Phase 3 — Ring animations + control-plane feedback
**Deliverable**: model-weight diamonds circulate all ring arcs; control-plane signals pulse from E2SAR back to EJFAT.

What to add:
- `cubicBez()` helper (copy from training-phase.html)
- **`startIntraNodeRing(node)`** — 3 diamonds per node, each traversing the 4-segment intra-node cubic-Bézier ring. `RING_DURATION=4.5s`, `repeat:-1`. Evenly phase-offset.
- **`startInterNodeRing()`** — 2 diamonds total, one traveling node1→node2 and one node2→node1 on the large inter-node arc. Same `gsap.to(state, {t, onUpdate})` pattern.
- **`schedControlFeedback(node, firstDelay)`** — small hollow circles travel from E2SAR Receiver left edge back to EJFAT right edge on the dashed control-plane path. Reschedules at `1.0 + random()*0.8s`. One instance per node, staggered.

**Inspect**: diamond motion on all ring segments (intra and inter-node), control circles traveling toward EJFAT, timing feel overall.

---

### Phase 4 — Polish and design refinements
**Deliverable**: final, screen-recording-ready animation.

#### Completed in this phase

- **SVG viewBox expanded** to `1440×845` to accommodate the legend without clipping Node 2
- **EVENT QUEUE relocated** inside E2SAR Receiver box (x=218–260, within E2SAR x=170–306); E2SAR→Queue arrow removed; Queue→Analyzer arrow lengthened from x=261 to x=374
- **Train-assembly timing fixed**: event packet appears in EVENT QUEUE only after the *last* packet of the train arrives (`arrivedCount === trainSize` guard)
- **`flashEl` re-entrancy bug fixed**: baseline stroke cached in `r.dataset.baseStroke/baseWidth`; `gsap.killTweensOf` + `gsap.set` reset before each flash, preventing border from growing inward
- **SHMEM→Proc packet color**: stays blue `#4a70a0` (was incorrectly purple)
- **AmSC MLFlow Model Registry block added** (x=1257, y=240, w=135, h=175); animated snapshot arcs from Rank 0 of each node via quadratic Bézier paths; version counter and flash animation match training-phase.html pattern
- **Rank labels**: PROC 0/1/2/3 renamed to Rank 0/1/2/3; lowercase "rank n" sub-labels removed; two-line layout "Rank n / training" centered in 60px box
- **MLFlow box cleaned up**: removed "Node 1 PROC 0" / "Node 2 PROC 0" source labels from inside the block
- **Legend redesigned**: 8 entries in 2-column × 4-row layout (Event data, Assembled Event, Processed Event, Model weights, MPI rank comms intra-node, Inter-node, EJFAT Control Plane, Model Snapshot); bounding box updated to match
- **Legend moved** to y=763 (15px lower) to clear Node 2 bottom edge (y=748)
- **Node labels** ("Perlmutter Node 1 / 2") increased to font-size=14, repositioned above node boxes (y=52 and y=417)
- **Container labels** ("ERSAP CONTAINER", "SAGIPS CONTAINER") increased by 2pt to font-size=12
- **Removed rotated labels**: "INTER-NODE", "MODEL WEIGHTS" removed from inter-node ring arc area
- **Subtitle removed**: "ERSAP container → shared memory → SAGIPS GPU processes · MPI model-weight ring" line deleted

- **Live frame timer**: `requestAnimationFrame` IIFE updates `id="frame-counter"` from `performance.now()` — ticking from page load
- **Packet color alternation**: Node 1 starts cyan (`#0099cc`), Node 2 starts orange (`#e05820`); `node.pktColor` captured per train and toggled after each event cycle so both senders' colors appear over time on both nodes
- **`index.html` gallery card added**: "Perlmutter Node Detail" card with static thumbnail SVG (EJFAT, two stacked node boxes, ERSAP pipeline, SAGIPS ranks, ring arcs, MLFlow)

**Inspect**: full end-to-end feel at screen-recording resolution (1440×845), readability of all counters, no jank or overlapping packets, MLFlow snapshot arcs visible from both Rank 0 sources.

---

**Phase 4 complete.** Remaining: git commit + PR for `feature/node-detail-animation`.

---

## Verification

1. Open `animations/node-detail.html` directly in browser (no server needed)
2. After 5-second delay, events begin flowing in both nodes (staggered ~1s apart)
3. Confirm: packet flies EJFAT→E2SAR, E2SAR flashes, queue depth increments/decrements, Analyzer flashes, SHMem Interface writes, SHMEM segment pulses, random SAGIPS proc highlights
4. Confirm: control feedback circles travel from each node's E2SAR back to EJFAT
5. Confirm: ring diamonds circulate both intra-node rings and inter-node arc continuously
6. Confirm: all text labels readable, no elements clipped, 16:9 layout intact at 1440×810
