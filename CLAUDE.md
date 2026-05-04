# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step or dependencies required. Open `index.html` directly in a browser, or serve locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`.

## Architecture

The entire game logic lives in a single file: `game.js` (vanilla ES6+, no frameworks). `index.html` is a thin shell that sets up an 800×600 `<canvas>` and loads the script.

**Game loop:** `requestAnimationFrame` calls `loop(ts)` each frame, which computes a delta-time `dt` (capped at 50 ms to prevent spiral-of-death on tab focus), then calls `update(dt)` and `draw()`.

**Entity classes** — each has `update(dt)`, `draw()`, and a `dead` boolean flag used for removal:
- `Ship` — player-controlled; handles thrust/rotation input, invincibility timer, shoot cooldown, and thruster flame rendering.
- `Asteroid` — spawns with a random irregular polygon (`verts`). `size` ∈ {1, 2, 3} maps to `RADII`/`SPEEDS`/`POINTS` arrays. `split()` returns two smaller asteroids.
- `Bullet` — fired from ship nose; has a TTL of 1.1 s and wraps around canvas edges.
- `Particle` — used for explosion effects; fades out via alpha based on remaining TTL ratio.

**Game state machine** — the `state` variable drives `update()` branching:
- `'playing'` — normal gameplay; runs all collision checks.
- `'dead'` — ship exploded, 2-second `deadTimer` before respawn; asteroids keep moving.
- `'gameover'` — all lives lost; Space key restarts via `initGame()`.

**Collision detection** is simple circle-circle (`dist()` helper). The bullet-vs-asteroid check marks both as `dead` and accumulates new fragments in `newAsteroids` before splicing the arrays. Ship collision uses a 0.82 shrink factor on the asteroid radius for a more forgiving hit box.

**Coordinate system:** origin top-left, x right, y down. `wrap(v, max)` handles toroidal wrapping on all moving entities.

**Input:** `keys` tracks held keys; `justPressed` tracks new-press-this-frame and is consumed by `pressed()` (returns true once per keydown).

## Controls

| Key | Action |
|-----|--------|
| `←` `→` | Rotate |
| `↑` | Thrust |
| `Space` | Shoot / Restart |

## Scoring

| Size | Points |
|------|--------|
| Large (3) | 20 |
| Medium (2) | 50 |
| Small (1) | 100 |
