# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Classic Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no `package.json`, no build step, no test suite, no linter. User-facing text (UI labels, overlay messages, README) is in Spanish.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve the folder statically and visit `http://localhost:8000`:

```bash
python -m http.server 8000
# or: npx serve .
```

Verification is manual: play in the browser.

## Architecture

Three files: `index.html` (DOM: `#board` canvas, side panel with `#score`/`#lines`/`#level`, `#next-canvas`, `#overlay` used for both PAUSA and GAME OVER), `style.css` (dark theme), and `game.js` (all logic, in plain global scope with `'use strict'`; no modules or classes).

Key points in `game.js`:

- **State** is a set of module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `lastTime`, `dropAccum`, `dropInterval`, `animId`), all reset in `init()`, which is also the restart button handler.
- **Board model**: `ROWS × COLS` matrix; `0` = empty, `1–7` = piece type, which doubles as the index into both `COLORS` and `PIECES`. Piece shapes store their type number in their cells, so merging copies the color index straight into the board.
- **Pieces** are `{ type, shape, x, y }`; `shape` is a copied square matrix. Rotation is `rotateCW` (transpose + reverse) with simple horizontal wall kicks `[0, -1, 1, -2, 2]` in `tryRotate` (no SRS tables, no counter-clockwise rotation).
- **`collide(shape, ox, oy)`** is the single source of truth for movement validity; rows with `ny < 0` are allowed (above the board).
- **Game loop** (`loop`) uses `requestAnimationFrame`, accumulates `dt` into `dropAccum`, and does gravity once it reaches `dropInterval`. Locking goes `lockPiece()` → `merge()` → `clearLines()` → `spawn()`; `spawn()` calls `endGame()` if the new piece collides immediately.
- **Scoring/levels** live in `clearLines()` (`LINE_SCORES[n] * level`, level = `floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`), plus +1 per soft-drop row and +2 per hard-drop cell.
- **Rendering**: `draw()` repaints the whole board every frame (grid → locked cells → ghost at `globalAlpha 0.2` → current piece). `drawNext()` runs only on spawn and assumes a 4×4 grid of 30px cells (120×120 canvas).

## Gotchas

- Canvas size in `index.html` (`300×600`) must equal `COLS*BLOCK × ROWS*BLOCK`; changing `COLS`, `ROWS` or `BLOCK` requires updating it.
- `togglePause()` shows the overlay on pause but does not hide it again on resume.
- `endGame()` cancels `animId`, but when it fires from inside `loop()` the loop then schedules a new frame anyway, so gravity keeps running after game over (input is blocked by `gameOver`).
