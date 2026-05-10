# CLAUDE.md — multiverse-screensaver

## What This Is
A single-file HTML5 canvas screensaver. No framework. No build step. No dependencies.
Pure vanilla JS rendering a multiverse of layered visual effects that shift between named dimensions.

**Repo:** `thebardchat/multiverse-screensaver`
**Deploy target:** GitHub Pages → Static HTML workflow (no build, files served as-is)
**Live file:** `trip.html` (rename to `index.html` for Pages root)

---

## Current State
- One file: `trip.html` — fully functional, visually complete
- Pushed to GitHub after local folder `multiverse-screensaver` was created
- **Known issue: renders slow** — diagnosed, root causes identified (see below)
- GitHub Pages not yet configured — this is the next step

---

## Project Goal (The Experiment)
Reverse the normal workflow:
1. Deploy to GitHub Pages first
2. Iterate live from the repo
3. Test performance fixes in the deployed version, not locally

This means: **GitHub Actions → Static HTML** is the correct workflow template. No Jekyll, no Astro, no build chain.

---

## Architecture
```
trip.html (single file)
├── Canvas setup + resize handler
├── Seeded RNG (XORshift)
├── 7 Dimensions (DIMS array) — named visual modes with bg colors + hue sets
├── Effects (11 total, 4-9 active at once, randomly picked per dimension shift)
│   ├── De Jong Strange Attractor   (DJ_N = 6000 points)
│   ├── Lorenz Attractor            (LOR_N = 3000 points)
│   ├── Particles + trails          (NP = 700, TRAIL = 18)
│   ├── Portals                     (5 portals, up to 12 rings each)
│   ├── Wormhole                    (55 rings, 90 pts each)
│   ├── Lissajous curves            (2400 pts)
│   ├── Fractal trees               (3 trees, depth 7, recursive)
│   ├── Reality Tears               (6 jagged glowing cracks)
│   ├── Nebulae                     (7 radial gradients, always on)
│   ├── Sine Grid                   (hypnotic wave interference)
│   └── Alien Geometry              (5-layer wobbling polygons)
├── Kaleidoscope overlay            (probabilistic, mirrors full canvas)
├── Dimension shift                 (auto every 10-24s, or on click)
└── Main loop via requestAnimationFrame
```

---

## Performance Issues — DIAGNOSED

### 1. 9,000 individual fillRect calls per frame (CRITICAL)
```js
// De Jong: 6,000 separate fillStyle + fillRect per frame
ctx.fillStyle=`hsla(${h},100%,65%,0.035)`;  // string built 6,000x
ctx.fillRect(x,y,1.5,1.5);

// Lorenz: 3,000 more of the same
```
**Fix:** Replace with `ImageData` / `putImageData` — write to pixel buffer directly, one GPU upload per frame.

### 2. shadowBlur inside nested loops (HIGH)
```js
// Portals: ~100+ shadow passes per frame (5 portals × rings × 3 colors)
ctx.shadowBlur=14;

// Fractal tree: every recursive branch() call
ctx.shadowBlur=7;

// Alien geometry: 5 layers
ctx.shadowBlur=18;
```
**Fix:** Remove `shadowBlur` from per-point/per-branch loops entirely. Use it only on full-layer overlays or drop it from attractors — glow on a 1.5px dot is invisible.

### 3. Kaleidoscope copies entire canvas every frame it's active
```js
offC.width=W; offC.height=H;       // resizes offscreen canvas every frame
offCtx.drawImage(canvas,0,0);       // full canvas copy
// draws back 6-8 times with clip  // effectively doubles render time
```
**Fix:** Only resize offscreen canvas on window resize. Cache it. Or disable kaleidoscope entirely for Phase 1 perf work.

### 4. Up to 9 effects running simultaneously
`particles` and `nebulae` are force-added on top of 4–9 random picks.
**Fix:** Hard cap at 5 active effects. Remove forced doubles.

### 5. Quick wins (zero visual impact)
- `DJ_N`: 6000 → 3000
- `LOR_N`: 3000 → 1500
- These are invisible improvements at normal scale

---

## File / Folder Structure (target)
```
multiverse-screensaver/
├── index.html          ← rename from trip.html for GH Pages root
├── CLAUDE.md           ← this file
└── .github/
    └── workflows/
        └── deploy.yml  ← Static HTML GitHub Actions workflow
```

---

## GitHub Pages Setup
Workflow file needed at `.github/workflows/deploy.yml`:
```yaml
name: Deploy static content to Pages
on:
  push:
    branches: ["main"]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: "pages"
  cancel-in-progress: false
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v5
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
Also enable GitHub Pages in repo Settings → Pages → Source: GitHub Actions.

---

## Constraints
- No frameworks. No npm. No build step. One HTML file.
- Do not add React, Vite, Webpack, or any bundler unless Shane explicitly asks.
- Performance fixes must not change the visual aesthetic — this is art, not a benchmark.
- `shadowBlur` can stay on portal rings and wormhole — remove only from attractor point loops and fractal recursion.
- Keep the seeded RNG (XORshift). Reproducibility matters for dimension states.

---

## Next Actions (in order)
1. Rename `trip.html` → `index.html`
2. Create `.github/workflows/deploy.yml` (see above)
3. Push → Pages auto-deploys
4. Apply performance fixes (ImageData for attractors, cap effects at 5, strip shadowBlur from loops)
5. Iterate live from repo

---

## ShaneBrain Context
- Part of `thebardchat` GitHub org
- Governed by the ShaneBrain Constitution
- This is an experiment project — low stakes, high creativity
- Log significant sessions via `shanebrain_log_conversation` mode: `CODE`
