# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file web Sudoku game (`sudoku.html`) with no build system, dependencies, or server. Open it directly in a browser to play.

## Development Workflow

- **Run:** Open `sudoku.html` directly in a browser (no server needed)
- **Test changes:** Refresh the browser after editing
- **Commit & push:** Use git + GitHub (`origin/master` on `github.com/Tokenekie/sudoku`)

## Architecture

Everything lives in `sudoku.html` — HTML structure, CSS, and JavaScript are all inline in one file. The JS is organized into clearly commented sections:

| Section | What it does |
|---|---|
| **State** | Six module-level arrays/vars (`board`, `solution`, `given`, `notes`, `history`, flags) represent the entire game state. All indices are flat 0–80. |
| **Geometry** | `rowOf`, `colOf`, `boxOf`, `peerSet` — pure functions for index math. No 2D arrays anywhere. |
| **Solver** | `canPlace`, `fillGrid` (backtracking fill with shuffle for randomness), `countSolutions` (stops at `limit=2` for uniqueness checks). |
| **Puzzle Generator** | `generatePuzzle(diff)` fills a complete grid then removes cells one-by-one, restoring any that break uniqueness. Target clue counts: easy=45, medium=35, hard=28. |
| **Render** | `buildCells()` rebuilds DOM once per new game. `render()` re-classifies existing cells on every state change — no virtual DOM. Box borders are applied via CSS `[data-col]`/`[data-row]` attribute selectors set during `buildCells`. CSS class order in the stylesheet determines highlight priority (peer < same-num < selected). |
| **Actions** | `inputNum`, `undoMove`, `hint`, `toggleNotes`, `toggleErrors` — each mutates state then calls `render()`. All user actions (keyboard + buttons) funnel through `inputNum(n)` where `n=0` means erase. |
| **Timer** | `setInterval` counter; cleared on win or new game. |

## Key Constraints

- The board is a flat `Array(81)` — cell `i` is at row `(i/9)|0`, col `i%9`.
- `render()` resets `el.className` from scratch every call (preserving `data-row`/`data-col` attributes). Adding new visual states means adding a CSS class and setting it inside `render()`.
- Puzzle generation runs on the main thread with a `setTimeout(..., 20)` deferral so the "Generating…" status message renders before the CPU-heavy backtracking begins. Hard puzzles can take 1–3 seconds.
- `history` is a stack of `{ i, v, ns }` snapshots — one entry per `inputNum` or `hint` call.
