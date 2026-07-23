# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris implementation using HTML5 Canvas. No build step, no dependencies, no package manager — just three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test tooling in this repo.

## Architecture

All game logic lives in `game.js` (~300 lines), organized around a few core concepts:

- **Board model**: `board` is a `ROWS × COLS` matrix (20×10). Each cell is `0` (empty) or an integer 1–7 identifying which piece color occupies it (see `COLORS`/`PIECES` arrays, index 0 unused).
- **Pieces**: each of the 7 tetrominoes is a square matrix in `PIECES`. `current` and `next` are piece objects `{ type, shape, x, y }`. Rotation (`rotateCW`) transposes + reverses rows rather than using precomputed rotation states.
- **Collision** (`collide`): checks board bounds and existing fixed blocks for a given shape/offset. Used for movement, rotation, and ghost-piece projection.
- **Wall kicks** (`tryRotate`): on rotation, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece down one row once `dropAccum >= dropInterval`.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears full rows (bottom-up scan with re-check of the same row index after a splice), then spawns the next piece.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/cell, soft drop 1 pt/row. Level increments every 10 lines cleared; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Rendering** (`draw`/`drawNext`): board is redrawn every frame — grid lines, locked blocks, ghost piece (`ghostY()`, drawn at `globalAlpha = 0.2`), then the current piece on top.
- **Game over**: detected in `spawn()` when a freshly spawned piece immediately collides; triggers `endGame()` and shows the overlay.

State is held in module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, etc.) reset by `init()`. There is a single `keydown` listener dispatching to movement/rotation/drop/pause actions.

## Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell pixel size), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
