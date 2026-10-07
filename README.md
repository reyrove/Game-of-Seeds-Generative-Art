# Game-of-Seeds-Generative-Art

A seed-based generative art catalogue for **Game of Seeds**, a computational cellular-automaton system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Game-of-Seeds-Generative-Art/`

---

## Overview

**Game of Seeds** renders a cellular automaton from numeric seeds, in the tradition of Conway's Game of Life but with a custom rule. Each composition seeds a grid with 10–30 three-cell groups, iterates a neighbour-sum rule 30–2030 times, and then draws the resulting state as a mosaic of glowing coloured dots and black dots on a dark ground.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical field.

---

## Features

### Generative Engine (p5.js)

- **Custom cellular automaton rule** — a dead cell comes alive if exactly 2 of its 8 neighbours are alive; all other cells die or stay dead:
  ```js
  if (e === 0 && [2].includes(a + b + c + d + f + g + h + i)) return 1;
  return 0;
  ```
- **Three-cell seed groups** — the grid starts with 10–30 three-cell groups placed at random positions, each either horizontal or vertical, with no overlap
- **Iteration range** — 30 to 2030 steps per composition
- **Grid size** — from 5 × 5 (coarse) to 104 × 104 (very fine), with the height drawn 5 cells off the width
- **Live cells** — drawn as coloured dots with a soft glow (`shadowBlur = cellW`), colour picked from a per-seed palette
- **Dead cells** — drawn as black dots
- **27 dark backgrounds** — including Black, Black Blue, Charcoal, Stormy Gray, Night Blue, Deep Teal, Dark Red, Midnight, Tomato, and others
- **46 curated palettes** — ranging from 2 to 8 foreground colours per palette
- **Edge-to-edge fill** — cells are centred in their grid positions with a radius of 97% of half a cell, so the composition fills the canvas with no cut-off circles

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Game-of-Seeds-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Game-of-Seeds-Generative-Art.git
   cd Game-of-Seeds-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — background colour, foreground palette, grid size, iteration count, group count, per-cell colours — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Background colour | 27 options (Black → Tomato) |
| Foreground palette | 46 options (2–8 colours each) |
| Grid width | 5 – 104 cells |
| Grid height | width − 5 to width + 5 |
| Total cells | 25 – ~11,000 |
| Iterations | 30 – 2030 |
| Seed groups | 10 – 30 |
| Cell radius | 97% of half-cell |

### The Custom Rule

The original sketch uses a rule written as:

```js
function rules(a, b, c, d, e, f, g, h, i) {
  if (e === 0 && [2].includes(a + b + c + d + f + g + h + i)) return 1;
  return 0;
}
```

Where `a` through `i` are the eight neighbours plus the cell itself in a 3 × 3 window:

```
a  b  c
d  e  f
g  h  i
```

So the rule reads: **a dead cell with exactly 2 live neighbours comes alive; every other cell (dead or alive) dies.**

This differs from Conway's Game of Life (where `e === 1` requires 2 or 3 neighbours to survive, and `e === 0` requires exactly 3 to birth). The simpler rule here produces sparser, more structured patterns — often branching, spiralling, or crystalline.

### The Three-Cell Seed

The grid starts empty. Then 10–30 three-cell groups are placed:

```js
const orientation = sr.int(0, 1);  // 0 = horizontal, 1 = vertical

// Horizontal: place at (row, col), (row, col+1), (row, col+2)
// Vertical:   place at (row, col), (row+1, col), (row+2, col)

// Each placement checks that all three target cells are empty;
// retries up to 200 times before giving up.
```

The three-cell groups give the automaton an initial "seed" to evolve from. Different positions, orientations, and counts produce radically different end states.

### The Iteration Loop

Each step replaces the entire grid with its successor:

```js
for (let step = 0; step <= numOfIt; step++) {
  for (let i = 1; i <= A1 - 2; i++) {
    for (let j = 1; j <= A2 - 2; j++) {
      // read a..i from cells
      // write result to cells1[j][i]
    }
  }
  cells = cells1.map(row => row.slice());  // swap buffers
}
```

The bounds (`i` from 1 to `A1 - 2`) leave a one-cell border, so neighbour reads never go out of range. With 30–2030 iterations, the pattern has time to develop complex structures before it stabilises or dies out.

### The Drawing — Centred, No Edge Cut-Off

The final state is drawn as a mosaic of circles. Each cell is placed at the **centre** of its grid cell, with a radius scaled from the smaller of the two cell dimensions:

```js
const cellW = cs / A1;
const cellH = cs / A2;
const r = Math.min(cellW, cellH) * 0.5 * 0.97;

for (let i = 0; i < A1; i++) {
  for (let j = 0; j < A2; j++) {
    const x = (i + 0.5) * cellW;
    const y = (j + 0.5) * cellH;

    if (cells[j] && cells[j][i] === 1) {
      const col = fgPalette[sr.int(0, fgPalette.length - 1)];
      p.fill(col);
      p.stroke(col);
      p.drawingContext.shadowColor = col;
      p.drawingContext.shadowBlur = cellW;
      p.ellipse(x, y, r * 2, r * 2);
    } else {
      p.fill(0, 0, 0);
      p.stroke(0, 0, 0);
      p.drawingContext.shadowBlur = 0;
      p.ellipse(x, y, r * 2, r * 2);
    }
  }
}
```

The `(i + 0.5) * cellW` positioning means the first column of dots sits half a cell from the left edge and the last column sits half a cell from the right edge — every circle fully visible, no cut-off, and the composition fills the canvas edge to edge.

The radius is 97% of half a cell, so adjacent circles have a small visible gap and read as a dense but distinct mosaic. If you want tighter packing, raise `0.97` to `1.0`. If you want more breathing room, lower it to `0.9`.

### The Glow

Live cells are drawn with a soft glow via the canvas shadow API:

```js
p.drawingContext.shadowColor = col;
p.drawingContext.shadowBlur = cellW;
```

This makes every coloured dot emit a faint halo in its own colour. Against the dark background, the effect reads as a bioluminescent or ornamental mosaic. Dead cells explicitly reset `shadowBlur = 0` so their black dots stay crisp.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the rule

Edit the `rules` function. The current rule is:

```js
if (e === 0 && [2].includes(a + b + c + d + f + g + h + i)) return 1;
return 0;
```

Common alternatives:

**Conway's Game of Life (survival + birth):**
```js
const sum = a + b + c + d + f + g + h + i;
if (e === 1 && (sum === 2 || sum === 3)) return 1;
if (e === 0 && sum === 3) return 1;
return 0;
```

**Pure birth rule:**
```js
const sum = a + b + c + d + f + g + h + i;
if (e === 0 && sum === 3) return 1;
return 0;
```

**A rule with 3 different birth/survival counts:**
```js
const sum = a + b + c + d + f + g + h + i;
if (e === 0 && (sum === 3 || sum === 4)) return 1;
if (e === 1 && sum >= 2) return 1;
return 0;
```

### Adjust the grid size range

```js
const widthDivision = sr.int(5, 104);
const heightDivision = widthDivision + sr.int(-5, 5);
```

- **Smaller ranges** (e.g., `sr.int(20, 40)`) produce chunkier cells and more legible patterns.
- **Larger ranges** (e.g., `sr.int(50, 150)`) produce finer mosaics but require more compute per composition.

### Adjust the iteration count

```js
const numOfIt = sr.int(30, 2029);
```

- **Lower values** (30–200) show the pattern mid-development — often structured but incomplete.
- **Higher values** (500–2030) show the pattern at a later stage — often dense, stable, or chaotic.

Very high iteration counts on a fine grid (e.g., 100 × 100 with 2000 iterations) take a moment to compute. If performance is a concern, cap the iteration count.

### Adjust the seed group count

```js
const numberOfGroup = sr.int(10, 29);
```

- **Fewer groups** (10–15) produce more isolated clusters that evolve independently.
- **More groups** (25–30) produce a denser starting field that evolves into a busier end state.

### Change the radius

```js
const r = Math.min(cellW, cellH) * 0.5 * 0.97;
```

- **1.0** — circles touch; maximal fill.
- **0.97** — small gap between circles (current).
- **0.85** — visible breathing room between circles.
- **0.6** — sparse dots that read as a scatter rather than a mosaic.

### Add a margin around the composition

To restore a small inset frame (one cell of margin on each side):

```js
const margin = 1;
const cellW = cs / (A1 + margin * 2);
const cellH = cs / (A2 + margin * 2);
const r = Math.min(cellW, cellH) * 0.5 * 0.97;

// Inside the loop:
const x = (i + 1 + margin) * cellW;
const y = (j + 1 + margin) * cellH;
```

### Adjust the glow strength

```js
p.drawingContext.shadowBlur = cellW;
```

- **0** — no glow; crisp circles.
- **cellW** — soft glow about the size of a cell (current).
- **cellW * 2** — a wide, diffuse bloom.

### Add a new palette

Extend the `FG_PALETTES` array:

```js
const FG_PALETTES = [
  // ... existing palettes
  ['#FF00FF', '#00FFFF', '#FFFF00'],  // your new palette
];
```

The seed will automatically pick it up on the next regeneration.

### Change the backgrounds

Extend the `BG_COLOURS` array:

```js
const BG_COLOURS = [
  // ... existing colours
  '#2A1B3D',  // your new background
];
```

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, Wandering Tiles, Food Grid, Framed Line, Deconstructed Stage, Layered Diagram, Japanese Patterns, Matrix Rain, Organic Shapes, Recursive Grid, and Generative Lifeform templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live automaton |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Game-of-Seeds-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **`$fx.rand()` replaced with `sr.rand()` / `sr.int()`** — the original sketch uses fxhash's `$fx.rand()`, which is not available in a plain browser. The seeded generator replicates the same deterministic behaviour, so the same seed produces the same composition.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original uses p5's global mode; here `p` is passed so the factory closure works.
6. **Drawing centred, no edge cut-off** — the original sketch places cells at `(i + 1) * cs / A1` with a radius derived from `cs / (2 * A1)`. That positions the first column one full cell from the edge and cuts off the last column. The adaptation positions each cell at `(i + 0.5) * cellW` with a radius of `Math.min(cellW, cellH) * 0.5 * 0.97`, so every circle is centred in its cell and fully visible against the canvas edges.
7. **`cells1` initialized as a copy** — the original uses `cells1 = cells` (aliasing the same array), which happens to work because the inner loop reads then writes each cell. The adaptation uses `cells1 = cells.map(row => row.slice())` on every iteration so the previous-generation state is never mutated mid-step.
8. **`attempts` guard added to `generateMatrixWithOnes`** — the original's `while (!placed)` loop can spin forever if too many groups are requested for a small grid. The adaptation caps retries at 200 per group.
9. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
10. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| [`Food-Grid-Generative-Art`](https://github.com/reyrove/Food-Grid-Generative-Art) | Illustrated food grid |
| [`Framed-Line-Generative-Art`](https://github.com/reyrove/Framed-Line-Generative-Art) | Gesture line composition |
| [`Deconstructed-Stage-Generative-Art`](https://github.com/reyrove/Deconstructed-Stage-Generative-Art) | Suprematist composition |
| [`Layered-Diagram-Generative-Art`](https://github.com/reyrove/Layered-Diagram-Generative-Art) | Layered linework composition |
| [`Japanese-Patterns-Generative-Art`](https://github.com/reyrove/Japanese-Patterns-Generative-Art) | Traditional motif composition |
| [`Matrix-Rain-Generative-Art`](https://github.com/reyrove/Matrix-Rain-Generative-Art) | Digital rain composition |
| [`Organic-Shapes-Generative-Art`](https://github.com/reyrove/Organic-Shapes-Generative-Art) | Layered petal-form composition |
| [`Recursive-Grid-Generative-Art`](https://github.com/reyrove/Recursive-Grid-Generative-Art) | Recursive subdivision composition |
| [`Generative-Lifeform-Generative-Art`](https://github.com/reyrove/Generative-Lifeform-Generative-Art) | Network / node-graph composition |
| **`Game-of-Seeds-Generative-Art`** | Cellular automaton composition *(this repo)* |

All seventeen share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Reference** — Conway's Game of Life (custom 3-cell rule)
- **Edition** — Game of Seeds, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**