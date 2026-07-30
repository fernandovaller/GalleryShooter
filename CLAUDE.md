# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

This is a single-file HTML game with no build step and no dependencies to install. It requires only an internet connection to load Phaser 3 from CDN.

```bash
# Serve with any static file server
python3 -m http.server 8000
# Then open http://localhost:8000
```

Or open `index.html` directly in a browser.

## Architecture

**Single-file game.** All code lives in `index.html` inside a single `<script>` tag. There are no external assets (images, audio files) — everything is generated procedurally in code.

### Scene flow (Phaser 3)

```
BootScene  → MenuScene → GameScene (parallel UIScene)
                          ↕
                    PauseScene / GameOverScene
```

- **BootScene** — Generates all textures procedurally via `Phaser.Graphics`: background (`bg`), six enemy sprites (`enemy_*`), and item glow circles (`item_*`). No external sprite sheets.
- **MenuScene** — Title screen. Starts the Web Audio context on first click (autoplay policy).
- **GameScene** — Main gameplay loop: enemy spawning, shooting, damage, round progression, item drops, scenery updates.
- **UIScene** — HUD overlay (HP bar, ammo, score, combo, round). Receives `gameScene` via `init(data)`.
- **PauseScene** — Overlay pause menu. Resumes `GameScene` and `UIScene` on ESC/P.
- **GameOverScene** — Displays final stats. Restarting stops UI and starts a new `GameScene`.

### Audio engine

`Audio` object uses the **Web Audio API** directly (no `<audio>` tags, no files). All SFX are synthesized:
- `shot()` — filtered noise burst
- `hit()`, `kill()`, `hurt()`, `innocent()` — oscillators with envelopes
- `item()`, `reload()` — tonal beeps
- `startAmbient()` / `slowOn()` / `slowOff()` — low drone that shifts during slow-mo
- `thunder()` — noise burst for lightning

The audio context must be resumed after browser suspension (calls in `create()` and click handlers).

### Enemy class hierarchy

All enemies extend `Phaser.GameObjects.Container`:

- `Enemy` (base) — handles visibility timer (`vis` range), silhouette entrance tween, escape/die logic, shadow, depth tinting.
- `Soldier`, `Sniper` — thin wrappers over base.
- `Tank` — overrides `takeHit()` to show HP bar (`this.bar.scaleX`).
- `Suicide` — overrides `preUpdate()` to charge upward (`this.y -= speed`) and explodes at `targetY`, dealing damage instantly.
- `Ghost` — phase-blinks (`alpha` toggles every 700ms); `takeHit()` returns `false` when not phasable.

`PowerUp` is also a Container with emoji text and a pulsing tween.

### Key constants (top of script)

```js
const BASE_W = 1280, BASE_H = 720; // internal resolution, scaled via Phaser.Scale.FIT
const MAX_ROUNDS = 10;             // rounds to win
const CLIP_SIZE = 30;              // ammo per clip
const RELOAD_MS = 1000;            // reload duration
```

Balance tables (`ENEMY_DEFS`, `ITEM_DEFS`) are also at the top.

### Procedural scenery (GameScene)

`buildScenery()` creates a multi-layer parallax scene with depth-based tinting:
- Depth 0: sky gradient, stars, moon
- Depth 1–3: back/mid building layers with blinking windows
- Depth 4: wire lines
- Depth 5 & 16: rain layers (back and front)
- Depth 14: steam particles
- Depth 15: front wall, props (barrels, boxes, vent, puddles)
- Depth 17: spotlights

Mouse parallax shifts `layerBack`, `layerMid`, `layerFront` by ratios in `updateScenery()`.
Lightning flashes are scheduled randomly and trigger `Audio.thunder()`.

### Important implementation notes

- **No external assets** — do not add image/audio file references. Use `graphics.generateTexture(key, w, h)` for new sprites.
- **Scoring** — `onEnemyKilled()` adds `e.def.pts * combo`. `grenade()` delegates to `die()` which already scores; do not double-count.
- **Innocents** — Shooting an innocent (`type === 'innocent'`) resets combo, penalizes score/HP, and flashes the damage vignette. At least 1 innocent spawns per round; count grows with `round/2`.
- **Slow motion** — `applySlow()` sets `enemy.slowFactor = 0.5`, which is read in `preUpdate()` and enemy movement code. Audio drone pitch drops via `Audio.slowOn()`.
- **Crosshair hit detection** — Uses simple AABB (`Math.abs(e.x - tx) < e.width/2 + 6`) rather than physics bodies. Iterates enemies in reverse for topmost priority.
- **Item collection** — Items are collected via the same click handler as shooting (`tryShoot()` checks `activeItem` bounds). This prevents double-collection since the cursor is hidden and there is one input handler.
