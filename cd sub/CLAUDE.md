# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`svg-particles.html` — single-file interactive canvas animation. No build step, no dependencies.

## Local server

```bash
python3 -m http.server 8080
# → http://localhost:8080/svg-particles.html
```

Required when using `SVG_FILE_PATH` (fetch doesn't work on `file://`). Not needed when using `SVG_STRING`.

## SVG input — two options (top of script)

```js
const SVG_STRING   = null;  // paste SVG markup directly
const SVG_FILE_PATH = null; // './file.svg' — needs local server
```

Supported SVG elements: `<circle>`, `<ellipse>`, `<rect>`. Each becomes one particle. `<path>` and `<polygon>` are intentionally excluded (stubs left in comments for extension).

## Architecture

All logic lives in a single `<script>` block, split into 10 numbered sections:

| Section | Responsibility |
|---|---|
| 1 `loadSVG()` | fetch file → inline string → built-in demo fallback |
| 2 `parseSVGPoints()` | DOMParser → extract center coords, normalise viewBox |
| 3 Canvas/transform | `resizeCanvas()`, `recalcSVGTransform()`, `svgToScreen()` — dpr-aware |
| 4 `Particle` class | position, velocity, spring params, visual type, phase, flash |
| 5 State management | `createParticles`, `setGatherTargets`, `setScatterTargets`, `triggerScatter`, `triggerGather` |
| 6 `updateParticles()` | spring force + noise jitter + mouse repulsion + tremble |
| 7 `renderParticles()` | semi-transparent black overlay (trail) + draw each particle |
| 8 `animate()` | rAF loop |
| 9 Utilities | `generateDefaultSVG()`, `setStatus()`, auto-loop |
| 10 `init()` | wires everything; auto-gathers after `CFG.autoGatherDelay` ms |

## Key config (`CFG` object)

```js
gatherSpringK / gatherDamping   // spring stiffness + damping while gathering
scatterSpringK / scatterDamping // same for scattering
noiseAmp        // per-frame random impulse (jitter feel)
trembleAmp      // residual oscillation when particle is settled at target
trailAlpha      // 0 = full clear, 1 = no trail; default 0.16
autoLoop        // false by default; set true to cycle automatically
```

Per-particle spring values are randomised ±~28% around CFG values to prevent lockstep movement.

## Resize handling

`window resize` → `resizeCanvas()` → `recalcSVGTransform()` → `setGatherTargets()`. If currently in gather state, `ctX/ctY` are immediately updated to new screen coords.
