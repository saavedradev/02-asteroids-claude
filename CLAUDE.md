# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

Open `index.html` directly in a browser, or use a local HTTP server:
```bash
npx serve .
```
Then visit `http://localhost:3000`.

## Architecture

Single-file game (`game.js`) using HTML5 Canvas, no bundler, no dependencies.

### Sections and data flow

**Game state** (lines 238–241): Global variables—`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` ('playing' | 'dead' | 'gameover').

**Classes** (lines 33–236):
- `Bullet` — projectile with TTL, wrapping edges
- `Asteroid` — irregular polygon, splits into 2 smaller when destroyed, size 1–3 determines radius and points
- `Ship` — player avatar, invincibility timer after spawn, shoot cooldown, thrust/rotation/drag physics
- `Particle` — explosion debris, fade-out over life span

**Input** (lines 8–24): `keys` object tracks continuous hold; `justPressed` tracks single frame trigger (used for Space to shoot, Space to restart). Consumed by `pressed()`.

**Update loop** (lines 293–351): Three states handle different gameplay phases. Ship/bullets/asteroids update position with wrapping. Collision: bullets vs asteroids (point award, split), ship vs asteroids (death). No asteroids → `nextLevel()`.

**Draw** (lines 396–409): Clear canvas, render particles/asteroids/bullets/ship, draw HUD (score/level/lives), overlay for game-over.

**Main loop** (lines 414–420): `requestAnimationFrame` with delta-time capping at 0.05s to prevent frame skips from breaking physics.

### Physics conventions

- Wrapping: `wrap(v, max)` handles toroidal edges for all moving objects
- Collision: distance-based, `dist()` computes Euclidean distance
- Velocity: `vx`/`vy` stored per object, applied each frame scaled by `dt`
- Ship: rotation speed 3.5 rad/s, thrust 260 px/s², drag 0.987 per frame

### Key behaviors

- **Asteroid spawn safety**: New asteroids spawn at least 130px from center to avoid instant collision
- **Invincibility blinking**: Ship flickers during 3-second invincibility (line 174), checked every `Math.floor(invincible * 8)` to avoid flicker aliasing
- **Level progression**: Each level spawns `3 + level` large asteroids
- **Ship death**: Triggers explosion particles, 2-second dead state, respawn if lives > 0
