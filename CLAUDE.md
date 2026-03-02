# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A collection of self-contained WebGL (OpenGL ES 2.0) shader demos. No build step, no dependencies, no package manager. Each demo is a single HTML file; `index.html` lists them all.

**To run:** open any `.html` file directly in a browser.

## Demos

| File | Effect |
|---|---|
| `mandelbrot.html` | Mandelbrot set zoom animation with interactive target selection |
| `warp.html` | Domain-warped FBM noise producing organic swirling flow |

## Shared architecture

Every demo follows the same structure:

1. **CSS overlay** — HUD and controls hint are `position: fixed` divs with `pointer-events: none` layered above the canvas.

2. **GLSL shaders** (inline JS strings) — a trivial vertex shader passes a fullscreen triangle-strip quad through unchanged; all visual logic is in the fragment shader.

3. **JS animation loop** — `requestAnimationFrame` loop uploads uniforms (`u_res`, `u_time`, plus demo-specific ones) and calls `gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4)`.

## Per-demo notes

### mandelbrot.html
- Fragment shader maps each pixel to `c = center + uv/zoom`, iterates `z → z² + c` up to 256 times, and applies smooth coloring `n + 1 − log₂(log₂|z|)` fed into an IQ cosine palette.
- `zoom` advances as `zoom *= exp(zoomRate * dt)`; `zoomRate` is signed so direction is reversible.
- Click handler inverts the shader UV mapping (`canvas.height * 0.5 - e.clientY` to flip Y) to convert mouse pixels → complex coordinates, updating `cx`/`cy` without resetting zoom.
- `highp float` limits useful zoom to `ZOOM_MAX = 6e5`; animation wraps at that boundary.
- The UV scale factor `2.5` is shared between the fragment shader and `mouseToComplex()` — changing one requires changing the other.

### warp.html
- Two-pass domain warping: `q = fbm(uv)`, `r = fbm(uv + q)`, `f = fbm(uv + r)`. Each extra pass folds noise back on itself for turbulent structure unreachable by single-pass noise.
- FBM uses a trig-free hash and a `mat2(1.6, 1.2, -1.2, 1.6)` rotation+scaling matrix between octaves (det = 4 → ×2 frequency per octave, matching the ×0.5 amplitude decay).
- Time is accumulated in JS as `timeAcc += dt * 0.001 * speed` and passed as `u_time`; pause sets `speed` to zero rather than stopping `requestAnimationFrame`.
