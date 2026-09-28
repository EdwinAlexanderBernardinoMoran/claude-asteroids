# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A clone of the classic arcade game **Asteroids**, implemented in pure HTML5 Canvas with no frameworks, bundler, or dependencies. The entire game logic lives in a single file: `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build step, package.json, linter, or test suite — it's plain static files (`index.html`, `game.js`, `favicon.svg`) served as-is.

## Architecture

`game.js` is a single self-contained file organized top-to-bottom into sections (marked with `── Section ──` comments):

- **Input** — `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (e.g. for firing) while `keys[code]` reflects held-down state (e.g. for rotation/thrust).
- **Utils** — `wrap` (toroidal position wrapping), `dist`, `rand`, `randInt`.
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`; entities mark themselves `dead = true` rather than removing themselves from arrays.
- **Game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) rather than a state object/class. `state` is one of `'playing' | 'dead' | 'gameover'`.
- **`update(dt)`** — branches on `state` first, then: reads input to fire, updates all entities, filters dead entities out of arrays, does bullet↔asteroid and ship↔asteroid collision (`dist(a, b) < combined radius`), splits destroyed asteroids into two smaller ones via `Asteroid.split()`, and advances to `nextLevel()` when `asteroids.length === 0`.
- **`draw()`** — clears canvas, draws particles → asteroids → bullets → ship (back-to-front), then HUD/overlays.
- **Main loop** — `requestAnimationFrame(loop)` computes `dt` (clamped to 0.05s max) and calls `update(dt)` then `draw()`.

Canvas is fixed at `W = 800`, `H = 600` (also set as `<canvas>` width/height attributes in `index.html`). All positions wrap toroidally via `wrap()`.

Asteroids have 3 sizes (index 1–3, small→large) with per-size arrays for radius (`RADII`), speed (`SPEEDS`), and score value (`POINTS`); splitting an asteroid of size N produces two of size N-1 (size 1 asteroids don't split).

When adding new entity types or game states, follow the existing pattern: a class with `update(dt)`/`draw()` and a `dead` flag, pushed into/filtered out of the relevant array in `update()`.
