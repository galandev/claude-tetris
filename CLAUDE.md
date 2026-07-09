# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package manager.

## Running / testing

There is no build, lint, or test tooling. To run the game, just open `index.html` in a browser, or serve the directory statically:

```bash
open index.html               # macOS, simplest option
python3 -m http.server 8000   # or any static server
```

Then verify changes manually in the browser (there is no automated test suite).

## Architecture

Three files, all at the repo root, no modules/bundler:

- `index.html` — DOM structure: a `#board` canvas (300×600, 10×20 cells at `BLOCK=30`px), a `#next-canvas` (120×120) for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and a `#overlay` div reused for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, single file, no classes, top-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`, ...).

If board dimensions or `BLOCK` size change in `game.js`, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).

### Core mechanics (all in `game.js`)

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index `1–7`.
- **Pieces**: `PIECES` array of square matrices (I, O, T, S, Z, J, L). `rotateCW` rotates via transpose + row-reverse.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and existing locked cells.
- **Rotation / wall kicks**: `tryRotate()` rotates then tries offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and drops the piece one row once `dropAccum >= dropInterval`.
- **Locking**: `lockPiece()` → `merge()` (bakes piece into `board`) → `clearLines()` → `spawn()` (next piece becomes current, new next piece generated). If the newly spawned piece immediately collides, `endGame()` fires.
- **Line clearing**: `clearLines()` scans bottom-up, splices full rows out and unshifts empty rows in; scoring uses `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
- **Scoring**: hard drop = 2 pts/row dropped, soft drop = 1 pt/row.

### Controls (keydown handler at bottom of `game.js`)

`ArrowLeft`/`ArrowRight` move, `ArrowUp`/`KeyX` rotate, `ArrowDown` soft drop, `Space` hard drop, `KeyP` pause (works even mid-input, checked before the paused/gameOver guard).

## Tunable constants (`game.js`, top of file)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`, initial `dropInterval` (in `init()`).
