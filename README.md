# pi-agent-tests

Playground for testing AI coding agents (pi) against local LLMs served with
[llama.cpp](https://github.com/ggml-org/llama.cpp). Each file is an artifact or
test case from a run; nothing to build, install, or configure.

## Contents

| File | What it is |
| --- | --- |
| [`solar-system.html`](solar-system.html) | Attempt 1 — top-down 2D sim with real J2000 ephemerides (~665 lines). |
| [`solar-system2.html`](solar-system2.html) | Attempt 2 — orbit-camera perspective 3D sim with simplified orbits (~486 lines). |
| [`llama.cpp.md`](llama.cpp.md) | `llama serve` recipes for Qwen3.8-27B-GGUF (MTP draft decoding, hardware variants), the prompt used to generate the sims, and per-config throughput notes. |

**Provenance:** both HTML files were generated in one shot from the same prompt
by pi running a local `Qwen3.8-27B-GGUF` via `llama serve` (see `llama.cpp.md`
for the commands and the exact prompt: *"write a solar system simulation that i
can run in my browser. single html file. no dependencies."*). They are two
independent answers to that prompt — useful as a side-by-side of what the model
does run to run.

| | `solar-system.html` | `solar-system2.html` |
| --- | --- | --- |
| View | fixed top-down, pseudo-3D | rotatable 3D (yaw/pitch, perspective projection) |
| Orbits | Keplerian, eccentric, J2000 phases → real positions for today | circular, hardcoded phases, compressed radii |
| Time | log-scale slider 0.1–1000 days/s, reset-to-today | linear slider 0–365 days/s, sim starts at "Year 1, day 0" |
| Body info | click → info panel | hover → tooltip |
| Detail | 3 moons, ~420 asteroids, fit-view + number-key focus | 1 moon (Earth's), ~700 asteroids, click-to-follow only |
| Style | layered (`scale` / `ephemeris` / app), modern JS | flat ES5-ish script, `var` throughout |

---

## solar-system.html

A top-down, pseudo-3D Solar System in one HTML file — Canvas 2D, zero
dependencies, ~665 lines.

### Run it

```bash
open solar-system.html
# or just double-click it
```

No build step, no server, works offline. Same for `solar-system2.html`.

### Features

- **Real orbital mechanics** — planets follow Keplerian orbits (solved via
  Newton iteration) with real semi-major axes, eccentricities, and periods.
  Phases use the J2000 epoch (mean longitude + longitude of perihelion), so
  the simulation starts with the planets **where they actually are today**.
- **All 8 planets + Sun**, with relative sizes, gas-giant bands, Saturn's
  rings, and display-only moons (Earth's Moon; Jupiter's Io, Europa, Ganymede).
- **Asteroid belt** — ~420 rocks on circular orbits, periods from Kepler's
  third law.
- **Camera** — drag to pan, scroll/pinch to zoom, click a planet to follow it
  as it orbits.
- **Time controls** — log-scale speed slider (0.1 days/s → 1000 days/s),
  pause/resume, reset to today.
- **Toggles** — orbit lines, labels, asteroid belt.
- **Info panel** — click any body for distance, year/day length, diameter,
  moon count, and a fun fact.

> Distances and sizes are compressed for display (orbit radius ∝ AU^0.55 so
> Neptune fits on screen; sizes scaled by √radius) — not to scale.

### Controls

| Input | Action |
| --- | --- |
| drag | pan |
| scroll / pinch | zoom |
| click planet | focus + info panel |
| click empty space | release focus |
| `Space` | pause / resume |
| `1`–`8` | focus Mercury → Neptune |
| `T` | reset to today |
| `0` / `F` | fit view |
| `Esc` | release focus |

### Internal structure

The script is organized in three layers (see the header comments in the file):

1. `scale` — the single display-scale policy (km → px, AU → px, moon spacing).
2. `ephemeris` — pure orbital mechanics; no DOM, no globals, no mutation.
3. App — canvas, camera/transforms, input, starfield, belt, rendering loop.

---

## solar-system2.html

A rotatable 3D Solar System in one HTML file — Canvas 2D with a hand-rolled
perspective projection, zero dependencies, ~486 lines.

### Features

- **Orbit camera** — yaw/pitch rotation by dragging, exponential wheel zoom
  (0.25×–40×), pitch clamped to 0.03–1.55 rad. Bodies are painter-sorted by
  depth; the Sun's ring halves are drawn front/back around Saturn.
- **Simplified orbits** — circular, with compressed radii and hardcoded start
  phases. Periods are real (in days), so relative orbital speeds are correct;
  absolute positions are not.
- **Follow mode** — click the Sun or any planet to have the camera target glide
  to it (exponential easing); `Esc`, empty-space click, or *Reset view* releases.
- **Rendering** — radial-gradient Sun with corona, per-planet lit/dark gradient
  shading toward the screen-space Sun direction, Jupiter/Saturn cloud bands +
  Great Red Spot (clipped to the disc), Saturn's rings, Earth's Moon,
  ~700 asteroids, ~650 twinkling stars.
- **Hover tooltips** — AU distance, year length, and a fun fact.
- **Touch** — one finger rotates, two fingers pinch-zoom.

### Controls

| Input | Action |
| --- | --- |
| drag / one finger | rotate (yaw + pitch) |
| scroll / pinch | zoom |
| hover body | tooltip |
| click body | follow it |
| click empty space / `Esc` | stop following |
| `Space` | pause / resume |

HUD: speed slider (0–365 days/s), pause, reset view, and toggles for orbit
lines, labels, and the asteroid belt.

---

## llama.cpp.md

Reference sheet for serving **`ggml-org/Qwen3.8-27B-GGUF`** locally with MTP
draft speculative decoding (`--spec-default --spec-type draft-mtp`), including:

- simple / advanced / 32 GB VRAM (RTX 5090) / DGX Spark / 16 GB laptop (RTX
  5080, ~60 t/s) command variants,
- reasoning-budget flags (`--reasoning-budget`, `--reasoning-budget-message`),
- the three `llama serve` configs used while generating the solar system sims,
  with measured throughput (11 → 16 → 13 tok/s) as knobs were tuned.

Minimal reproduction:

```bash
llama serve -hf ggml-org/Qwen3.8-27B-GGUF --spec-type draft-mtp
```

Then point your client at the OpenAI-compatible endpoint and send the prompt
recorded in the file.