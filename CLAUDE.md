# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

There is no build/lint/test tooling. To run:

```bash
start index.html        # Windows: open directly in a browser
```

Or serve it locally (needed if a feature requires `http://` instead of `file://`):

```bash
python3 -m http.server 8000
npx serve .
```

Then open `http://localhost:8000`.

## Architecture

Everything lives in `game.js` (~300 lines), organized around a handful of global `let` bindings that hold game state: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`.

- **Board model**: `board` is a `ROWS × COLS` matrix (20×10). Each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices (color index in place of `1`). `rotateCW` rotates a shape via transpose + row-reverse.
- **Collision** (`collide`): checks a shape at a given offset against board bounds and already-locked cells.
- **Wall kicks** (`tryRotate`): rotates the current piece, then on collision retries at offsets `[-1, 1, -2, 2]` before giving up.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded, otherwise locks it.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (scanning bottom-up, re-checking the same row index after a splice), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points/row dropped, soft drop adds 1 point/row.
- **Leveling**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece already collides.

Rendering is plain Canvas 2D (`drawBlock`, `drawGrid`, `draw`, `drawNext`) — no game framework. Input is a single `keydown` listener mapping arrow keys / `X` / `Space` / `P` to movement, rotation, soft/hard drop, and pause.

`index.html` just declares the DOM shell (board canvas, next-piece canvas, score/lines/level panel, pause/game-over overlay) and loads `game.js`. `style.css` provides the dark/retro visual theme.

## Tuning constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`, `ROWS`, or `BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
