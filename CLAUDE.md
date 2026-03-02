# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A self-contained WebGL (OpenGL ES 2.0) demo — `mandelbrot.html` — that renders an animated Mandelbrot set zoom with interactive controls. No build step, no dependencies, no package manager.

**To run:** open `mandelbrot.html` directly in a browser.

## Architecture

Everything lives in a single HTML file with three logical sections:

1. **CSS overlay** — HUD, controls hint, reticle, and click-marker are positioned-fixed divs layered above the canvas via `pointer-events: none`.

2. **GLSL shaders** (inline JS strings)
   - Vertex shader: passes a fullscreen triangle-strip quad through unchanged.
   - Fragment shader: maps each pixel to a complex number `c = center + uv/zoom`, runs the Mandelbrot recurrence `z → z² + c` for up to 256 iterations, applies smooth/continuous coloring (`n + 1 − log₂(log₂|z|)`) to eliminate banding, and feeds the result into an IQ cosine palette animated by `u_time`.

3. **JS animation loop**
   - `zoom` advances each frame as `zoom *= exp(zoomRate * dt)` (signed rate → reversible direction).
   - Click handler inverts the fragment shader's UV mapping to convert mouse pixels → complex coordinates and updates `cx`/`cy` without resetting zoom.
   - Speed, direction, and reset are keyboard/scroll-wheel controlled.

## Key constraints

- `highp float` (32-bit) limits useful zoom depth to ~`6e5` before precision artifacts appear; the animation wraps at `ZOOM_MAX = 6e5`.
- The UV scale factor `2.5` in the fragment shader means the shorter screen dimension spans ±1.25 complex units at `zoom = 1`. Any change to this constant must be mirrored in `mouseToComplex()` in JS.
- Mouse Y must be flipped (`canvas.height * 0.5 - e.clientY`) when inverting the GL coordinate system.
