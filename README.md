# Minesweeper

A browser-based implementation of the classic Minesweeper game. The entire game — board, rendering, 3D bevelled cells, timer, mine counter, cascade reveal, chord click — lives in a single self-contained HTML file.

## Play it

Live on GitHub Pages: **https://geda.github.io/minesweeper/**

Or open `index.html` locally in any modern browser. No server, no build step, no dependencies.

## Files

- **`index.html`** — the game (HTML + inline CSS + inline JavaScript).
- **`minesweeper-rules.md`** — the full game specification used as input: grid, cell states, adjacency, mine counter, timer, controls, first-click safety, cascade/flood-fill, chord click, win and lose conditions, and edge cases.

## Then and now

I first wrote Minesweeper about **30 years ago, as a student, in Turbo Pascal**. Back then there was no UI framework to help — every cell, every bevelled border, every 3D shadow effect had to be drawn by hand in code. It took me roughly **three weeks** to get a playable version working.

This version was generated **100% by AI, in under two minutes**, from the specification in `minesweeper-rules.md`.

Same game. Three weeks → two minutes.
