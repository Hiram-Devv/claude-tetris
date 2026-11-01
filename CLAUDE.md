# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no tests, no linter. The README and all UI text are in Spanish — keep user-facing strings in Spanish.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve the folder statically:

```
python -m http.server 8000   # then http://localhost:8000
npx serve .
```

There is no automated test suite; verify changes by playing in the browser.

## Architecture

All logic lives in `game.js` as a single non-module script (`'use strict'`, global functions and mutable top-level `let` state). `index.html` loads it at the end of `<body>`, and it grabs DOM elements by id at load time — renaming an id in `index.html` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) requires updating `game.js`.

- **Board**: `ROWS × COLS` matrix; `0` = empty, `1–7` = piece type, which doubles as the index into both `COLORS` and `PIECES`. Piece shape matrices store their own type number in filled cells, so `merge()` copies the value straight onto the board.
- **Core loop**: `loop()` (requestAnimationFrame) accumulates `dt` into `dropAccum`; on reaching `dropInterval` it moves the piece down or calls `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` calling `endGame()` on immediate collision is the only game-over path.
- **Movement/rotation** all go through `collide(shape, x, y)`. Rotation is `rotateCW` (no SRS) with kicks `[0, -1, 1, -2, 2]` on x only, in `tryRotate()`.
- **Scoring/level**: `LINE_SCORES[cleared] * level`; hard drop +2/cell, soft drop +1/row. Level and `dropInterval` (`max(100, 1000 - (level-1)*90)`) are recomputed only inside `clearLines()`; the initial `1000` is also hardcoded in `init()`.
- **Pause/game over** share one overlay (`#overlay`, toggled with the `hidden` class) whose title/text is rewritten per state; pausing cancels the rAF via `animId`, resuming calls `loop()` directly after resetting `lastTime`.
- `init()` fully resets state and is also the restart button handler.

## Coupled values

- Changing `COLS`, `ROWS`, or `BLOCK` requires updating `<canvas id="board" width height>` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).
- `drawNext()` assumes a 4×4 grid of 30px cells (`NB = 30`), matching the 120×120 `#next-canvas`.
- Adding a piece type means extending `COLORS` and `PIECES` together and updating the `* 7` in `randomPiece()`.
