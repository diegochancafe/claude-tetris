# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no package.json, no build step, no test suite, no linter. The README and all user-facing text (UI labels, overlay messages, button text) are in Spanish. Keep new UI strings in Spanish.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve the folder statically and visit http://localhost:8000:

    python -m http.server 8000
    # or: npx serve .

Verify changes manually in the browser. There are no automated tests.

## Architecture

All game logic lives in `game.js`: a single script loaded with a plain `<script>` tag (not a module) in `'use strict'` mode. State is kept in module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...). `init()` resets them all, and the restart button calls it too.

Key invariants that span several functions:

- **Piece type = color index = cell value.** `PIECES[t]` matrices hold the value `t` in their filled cells, `COLORS[t]` is that piece's color, and `board` cells store `0` (empty) or that `t`. Index 0 of both arrays is `null`. A new piece must follow this scheme.
- **Collision is the single source of truth.** `collide(shape, ox, oy)` is used for movement, rotation (`tryRotate` tries the kicks `[0,-1,1,-2,2]`), gravity, ghost projection (`ghostY`) and game-over detection (`spawn` collides immediately → `endGame`). Cells with `y < 0` are allowed (above the board).
- **Game loop:** `loop` runs on `requestAnimationFrame` and adds frame deltas to `dropAccum`; when that reaches `dropInterval` the piece falls one row or `lockPiece()` runs (`merge` → `clearLines` → `spawn`). Pausing and game over stop the loop with `cancelAnimationFrame(animId)`, and unpausing restarts it by calling `loop` directly.
- **Speed/level:** `level = floor(lines/10) + 1` and `dropInterval = max(100, 1000 - (level-1)*90)`. Both are recomputed only inside `clearLines`.
- **Rendering:** `draw()` repaints everything each frame in this order: grid, locked board, ghost (alpha 0.2), current piece. `drawNext()` runs only from `spawn()`. The HUD (`updateHUD`) updates only on events, never per frame.

## Coupled values (change together)

- `COLS × BLOCK` and `ROWS × BLOCK` in `game.js` must match the `width`/`height` of `<canvas id="board">` in `index.html` (currently 300×600).
- `drawNext()` hardcodes a 30px block size and a 4×4 cell area, which matches `<canvas id="next-canvas" width="120" height="120">`.
- Key bindings live in the `keydown` handler at the bottom of `game.js`. The controls list in `index.html` and the table in `README.md` describe the same keys, so update all three together.

## Known quirk

`togglePause()` shows the overlay when pausing but never hides it again (it never adds `hidden` back) when unpausing. Keep this in mind when touching pause/overlay logic.
