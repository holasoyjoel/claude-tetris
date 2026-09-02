# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running / verifying

No build, no tests, no linter, no dependencies — intentionally. Do not add `package.json`, a bundler, or a transpiler.

- Run: open `index.html` directly, or serve statically (`python3 -m http.server 8000`) and open `http://localhost:8000`.
- Verify changes by playing in a browser; there is no automated test path.

## Architecture

Three cooperating files: `index.html` (DOM + two canvases), `style.css` (dark theme), `game.js` (all logic, ~300 lines).

`game.js` is a single flat script (`'use strict'`, no IIFE/modules). All state lives in module-level `let` (`board, current, next, score, lines, level, paused, gameOver, dropInterval, ...`) mutated in place. `init()` runs at load and doubles as restart (wired to `#restart-btn`).

Key pieces:
- **Board model**: `ROWS×COLS` matrix; each cell is `0` (empty) or a color index `1..7`. That index maps into *both* `COLORS` and `PIECES`, and each piece's shape matrix is filled with its own type number. `COLORS` and `PIECES` must stay index-aligned — this shared index is the core invariant.
- **`collide(shape, ox, oy)`** is the single collision gate used by horizontal move, rotation, ghost projection, gravity, and spawn-time game over.
- **Rotation**: `rotateCW` (transpose + row-reverse) plus `tryRotate` basic wall kicks `[0, -1, 1, -2, 2]`. No SRS kick tables.
- **Game loop**: `requestAnimationFrame(loop)` accumulates `dropAccum` against `dropInterval`; a step drops one row or calls `lockPiece()` (merge → clearLines → spawn). Pause cancels the frame and resets `lastTime` on resume — anything added to the loop must tolerate that stop/restart.
- **Scoring / level**: `score += LINE_SCORES[cleared] * level`; `level = floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)`. Hard drop +2/cell, soft drop +1/row.
- **Rendering**: `draw()` redraws everything each frame. `drawBlock` is shared by the board canvas and the next-piece canvas via a size param; ghost piece drawn at alpha `0.2`.

## Coupling gotchas

- `COLS` / `ROWS` / `BLOCK` in `game.js` must match `<canvas id="board" width height>` in `index.html` (width = `COLS*BLOCK`, height = `ROWS*BLOCK`). `drawNext` hardcodes a 4×4 layout at `NB=30` against `#next-canvas` 120×120.
- `game.js` looks up all element IDs unguarded at load. Renaming an ID in `index.html` breaks the game silently at startup.
- `style.css` defines the `.hidden` class that `game.js` toggles on `#overlay`.

## Conventions

User-facing strings are Spanish (overlay text, README). Keep new UI text Spanish.
