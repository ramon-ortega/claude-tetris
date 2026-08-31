# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla-JS Tetris rendered on an HTML5 canvas. **No build step, no package.json, no dependencies, no tests.** Three source files: `index.html`, `style.css`, `game.js`.

## Running

```bash
open index.html              # works directly from the filesystem
python3 -m http.server 8000  # or serve statically, then open localhost:8000
```

There is nothing to install, compile, lint, or test. Verify changes by loading the page and playing; check the browser console for errors.

## Architecture (`game.js`)

Single top-level script (`'use strict'`, no modules) with module-level mutable state — `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId` — reset by `init()`. `init()` runs on load and is also the restart-button handler.

Key invariant: **cell value = piece type = color index**. Each entry in `PIECES` is filled with its own 1–7 index (the T piece's matrix contains `3`s), the board stores those same numbers, `0` means empty, and `COLORS[i]` maps index to color. Any new piece must keep its matrix filled with its own index, and `COLORS`/`PIECES` must stay aligned (both are 1-indexed with a leading `null`).

Rotation is transpose-and-reverse (`rotateCW`) on square-ish matrices; `tryRotate` attempts x-offsets `[0, -1, 1, -2, 2]` — a simplified wall kick, not the SRS kick tables.

Game loop: `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum` and drops one row when it exceeds `dropInterval`. Pausing/game-over both `cancelAnimationFrame(animId)`; unpausing resets `lastTime` before restarting the loop so the accumulated wall-clock time isn't applied as a burst of drops. Any new code path that suspends the loop must do the same.

Rendering redraws the whole board every frame: `drawGrid()` → settled cells → ghost (alpha 0.2, position from `ghostY()`) → current piece. HUD (`score`/`lines`/`level` spans) is updated separately via `updateHUD()`, which must be called after any state mutation — the keydown handler calls it once at the end for all inputs.

Scoring: `LINE_SCORES[cleared] * level`, plus 2/cell for hard drop and 1/row for soft drop. Level is `floor(lines / 10) + 1`; speed is `max(100, 1000 - (level - 1) * 90)` ms.

## Cross-file coupling

- Canvas dimensions are hardcoded in `index.html` (`board` = 300×600, `next-canvas` = 120×120). Changing `COLS`, `ROWS`, or `BLOCK` in `game.js` requires updating the `width`/`height` attributes to match `COLS*BLOCK` × `ROWS*BLOCK`. `drawNext()` also assumes a 4×4 grid at 30px.
- `game.js` grabs DOM nodes by id at script load: `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. Renaming an id in the HTML breaks the script silently at load.
- The overlay is toggled purely via the `.hidden` class in `style.css`.

## Language

UI strings, README, and code comments are in Spanish. Keep user-facing text in Spanish; identifiers are in English.
