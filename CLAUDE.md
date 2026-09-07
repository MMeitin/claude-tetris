# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris. No build step, no dependencies, no package.json. Three files:

- `index.html` — DOM structure, `<canvas id="board">` (300×600) and `<canvas id="next-canvas">` (120×120) for the next-piece preview.
- `style.css` — dark/retro arcade theme.
- `game.js` — all game logic (~300 lines).

## Running / testing

No build or test tooling exists. To run the game, open `index.html` directly in a browser, or serve it statically (e.g. `python3 -m http.server 8000`, `npx serve .`). There is no automated test suite — verify changes by playing the game in a browser.

## Architecture

Everything lives in `game.js` as module-level state and free functions (no classes, no framework):

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` defines each tetromino as a square matrix of color indices. `randomPiece()` picks one and centers it at spawn. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table.
- **Collision** (`collide`): checks board bounds and overlap with locked cells; used for movement, rotation, ghost-piece projection, and spawn (game-over check).
- **Wall kicks** (`tryRotate`): after rotating, tries horizontal offsets `[0, -1, 1, -2, 2]` in order and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded; `paused`/`gameOver` stop the loop via `cancelAnimationFrame`.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (scanned bottom-up, with the scan index incremented again after a splice so it re-checks the row that shifted down), then spawns the next piece.
- **Scoring/leveling**: `LINE_SCORES = [0,100,300,500,800]` scaled by `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. `level` increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering** (`draw`): clears and redraws the full canvas each frame — grid lines, locked board cells, the ghost piece (`globalAlpha = 0.2`, position from `ghostY()`), then the current piece on top. `drawNext()` renders the preview canvas the same way via `drawBlock`.

Tunable constants at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` × `ROWS×BLOCK`).

## CI / workflows

Three GitHub Actions workflows use `anthropics/claude-code-action@v1`: `claude.yml` responds to `@claude` mentions in issues/PRs, `claude-code-review.yml` reviews PRs automatically, and `claude-issue-triage.yml` runs on every issue opened/edited to auto-assign labels and post a Spanish-language technical diagnosis as a comment (skipped if the issue has the `no-triage` label).
