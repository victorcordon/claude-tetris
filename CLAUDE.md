# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no test suite, no linter. User-facing text (UI labels, overlay messages, README) is in Spanish — keep new strings consistent with that.

## Running

Open `index.html` directly in a browser, or serve the directory statically (preferred):

```bash
python -m http.server 8000   # then open http://localhost:8000
```

Verification is manual: play the game in a browser.

## Architecture

All game logic lives in `game.js` as top-level functions sharing module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of it and is also the restart handler. The script is loaded at the end of `<body>` and grabs DOM elements by id at load time, so ids in `index.html` and `game.js` must stay in sync.

Key conventions that span multiple functions:

- **Piece type = color index = cell value.** `PIECES[t]` shapes are filled with the value `t` itself (not `1`), so merging a piece into `board` writes its color index directly and `COLORS[t]` renders it. Index `0`/`null` means empty. When adding or reordering pieces, update `PIECES` and `COLORS` together and keep the `Math.random() * 7` in `randomPiece()` in sync.
- **Shapes are square matrices**; rotation is `rotateCW` (transpose + reverse) and `tryRotate` applies simple horizontal wall kicks `[0, -1, 1, -2, 2]` (not SRS).
- **`collide(shape, x, y)`** is the single source of truth for legality (movement, rotation, ghost, spawn/game over). Rows with `y < 0` are allowed above the board.
- **Piece lifecycle:** gravity in `loop()` / `softDrop()` / `hardDrop()` → `lockPiece()` → `merge()` → `clearLines()` (updates score, level, `dropInterval`) → `spawn()` (promotes `next`, triggers `endGame()` if the new piece collides).
- **Game loop** is `requestAnimationFrame`-driven with a time accumulator (`dropAccum` vs `dropInterval`). Pause and game over stop it via `cancelAnimationFrame(animId)`; resuming re-enters `loop()` after resetting `lastTime`.
- **Rendering** redraws the whole board every frame (`draw()`); the next-piece preview (`drawNext()`) is only redrawn on `spawn()` and assumes a 4×4 grid of 30px cells on the 120×120 `#next-canvas`.

## Gotchas

- Canvas size in `index.html` (`#board` 300×600) must equal `COLS × BLOCK` by `ROWS × BLOCK`; changing those constants requires updating the HTML.
- The single `#overlay` element is shared by the PAUSA and GAME OVER states.
