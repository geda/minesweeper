# Minesweeper — Complete Game Specification

## 1. Overview

Minesweeper is a single-player logic puzzle game played on a rectangular grid of cells.
Hidden mines are scattered across the board. The player reveals cells using numbered clues
to deduce which cells are safe, and avoids clicking on any mine.

---

## 2. Grid & Difficulty Levels

The board is a rectangular grid of cells. Three standard difficulty levels exist:

| Difficulty   | Grid Size | Mine Count | Mine Density |
|--------------|-----------|------------|--------------|
| Beginner     | 9 × 9     | 10         | ~12%         |
| Intermediate | 16 × 16   | 40         | ~16%         |
| Expert       | 30 × 16   | 99         | ~21%         |

A **Custom** mode allows any grid size up to 30 × 24, with a minimum of 10 mines and
a maximum of `(cols - 1) × (rows - 1)` mines.

---

## 3. Cell States

Each cell on the board is always in exactly one of three states:

| State      | Description                                                   |
|------------|---------------------------------------------------------------|
| `COVERED`  | Default state. Hidden, not yet interacted with.               |
| `REVEALED` | The cell has been opened. Shows a number (1–8) or is blank.   |
| `FLAGGED`  | Marked by the player as a suspected mine. Still considered covered (cannot be accidentally revealed). |

A flagged cell can also be in a sub-state:
- **Flag** (`🚩`): player is confident it is a mine.
- **Question mark** (`?`): player is uncertain (optional, toggled by a second right-click).

---

## 4. Cell Content (hidden until revealed)

Each cell contains one of:

| Content     | Description                                                              |
|-------------|--------------------------------------------------------------------------|
| **Mine**    | If revealed, the game is lost immediately.                               |
| **Number**  | An integer from 1 to 8, indicating the exact count of mines in the 8 adjacent neighbors. |
| **Blank**   | Zero adjacent mines (displayed as empty). Triggers an automatic cascade reveal. |

---

## 5. Adjacency

Each cell has up to 8 neighbors: horizontally, vertically, and diagonally adjacent cells.

```
[ ][ ][ ]
[ ][X][ ]   ← X has 8 neighbors (all surrounding cells)
[ ][ ][ ]
```

**Edge and corner cells have fewer neighbors:**
- Corner cells have **3** neighbors.
- Edge cells (non-corner) have **5** neighbors.
- All other cells have **8** neighbors.

The number displayed on a revealed cell counts **all** mines among its valid neighbors.
This count is always 100% accurate.

---

## 6. Mine Counter

A counter is displayed showing:

```
mine_counter = total_mines - number_of_flagged_cells
```

- Starts at the total mine count.
- Decreases by 1 each time the player places a flag.
- Increases by 1 each time the player removes a flag.
- **Can go negative** if the player places more flags than there are mines.
- This counter is informational only — flags may be placed on non-mine cells.

---

## 7. Timer

- The timer starts on the **player's first click** (not on game load).
- The timer stops when the player **wins** or **loses**.
- The timer counts elapsed seconds.
- Best times are recorded per difficulty level.

---

## 8. Controls

| Action                  | Input                            | Effect                                                                 |
|-------------------------|----------------------------------|------------------------------------------------------------------------|
| Reveal a cell           | Left-click on a `COVERED` cell   | Opens the cell. Shows number, blank, or mine.                          |
| Flag / unflag a cell    | Right-click on a `COVERED` cell  | Cycles: no flag → flag → question mark → no flag                       |
| Chord click             | Left-click on a `REVEALED` number | If the number of adjacent flags equals the cell's number, reveals all adjacent non-flagged covered cells. |
| New game                | Click the face/emoji button      | Resets the board.                                                      |

> **Note:** Left-clicking a `FLAGGED` cell does nothing (protection against accidental reveal).
> The player must remove the flag first.

---

## 9. First Click Safety Rule

The **first click is always safe** — it will never reveal a mine.

Implementation options:
- **Option A (pre-generated):** Generate the board before the first click, guaranteeing the clicked cell and its neighbors are mine-free.
- **Option B (post-generated):** Place mines only after the first click, excluding the clicked cell (and optionally its 8 neighbors) from mine placement.

**Cascade guarantee:** Most implementations also ensure the first click opens a blank cell (0 adjacent mines), giving the player a useful starting area. This is optional but strongly recommended.

---

## 10. Reveal Mechanics

### 10.1 Revealing a number cell
The cell transitions from `COVERED` to `REVEALED` and displays its number (1–8).
No other cells are affected.

### 10.2 Revealing a blank cell (cascade / flood-fill)
When a blank cell (0 adjacent mines) is revealed, the game **automatically reveals all its neighbors**.
This process is **recursive**: if any neighbor is also blank, its neighbors are revealed too,
continuing until all reachable blank cells and their bordering number cells are uncovered.

```
Cascade algorithm (flood-fill / BFS or DFS):
1. Reveal the clicked blank cell.
2. For each of its 8 neighbors:
   a. If COVERED and not FLAGGED:
      - Reveal it.
      - If it is also blank → recurse (add to queue).
      - If it is a number → reveal but do not recurse.
3. Stop when no more blank cells remain in the queue.
```

Flagged cells are **never** automatically revealed during a cascade.

### 10.3 Revealing a mine
The game ends immediately (see Section 12 — Lose Condition).

---

## 11. Chord Click (Advanced Mechanic)

A chord click is performed by **left-clicking on an already-revealed number cell**.

**Condition:** The number of adjacent `FLAGGED` cells equals the cell's displayed number.

**Effect:** All adjacent `COVERED` and non-flagged cells are revealed simultaneously.

**Risk:** If any of the adjacent flags are incorrectly placed (on a non-mine cell),
the chord click will reveal a mine and the game is lost.

---

## 12. Win Condition

The player wins when **all non-mine cells have been revealed**.

```
win = (number of REVEALED cells) == (total cells - total mines)
```

- Flagging mines is **not required** to win.
- When the win condition is met, all unflagged mines are automatically flagged.
- The timer stops.
- The face/emoji displays a win state (e.g., 😎).

---

## 13. Lose Condition

The player loses when a **mine cell is revealed** (by direct left-click or via chord click).

**On loss:**
1. The timer stops.
2. The clicked mine is highlighted (e.g., shown in red).
3. All other mines are revealed (shown as mines).
4. All incorrectly placed flags (flags on non-mine cells) are shown with an ❌ marker.
5. The face/emoji displays a lose state (e.g., 😵).
6. No further interaction is possible until the board is reset.

---

## 14. Board Generation Rules

1. Mines are placed **randomly** across the grid.
2. The first-clicked cell (and optionally its 8 neighbors) must **not** contain a mine.
3. No cell may contain more than one mine.
4. The maximum number of mines is `(cols - 1) × (rows - 1)`.

---

## 15. Number Color Convention (classic Windows style)

The number colors in the original Microsoft Windows Minesweeper are:

| Number | Color         |
|--------|---------------|
| 1      | Blue          |
| 2      | Green         |
| 3      | Red           |
| 4      | Dark Blue / Navy |
| 5      | Dark Red / Maroon |
| 6      | Cyan / Teal   |
| 7      | Black         |
| 8      | Gray          |

---

## 16. UI Elements Summary

| Element            | Description                                                    |
|--------------------|----------------------------------------------------------------|
| Grid               | The main play area of N × M cells.                            |
| Mine counter       | 3-digit display (top-left). Shows remaining unflagged mines.  |
| Timer              | 3-digit display (top-right). Counts seconds from first click. |
| Reset button       | Face/emoji in the center top. Resets the game.                |
| Face states        | 🙂 default · 😮 while holding click · 😎 win · 😵 lose        |

---

## 17. Edge Cases & Implementation Notes

| Case | Behaviour |
|------|-----------|
| Right-clicking a revealed cell | No effect. |
| Left-clicking a flagged cell | No effect (flag protects the cell). |
| Chord click when flag count < cell number | No effect. |
| Chord click when flag count > cell number | No effect. |
| Chord click that hits a mine | Game over (lose condition triggered). |
| Mine counter below 0 | Allowed — display negative value. |
| First click on corner/edge | Safe; cascade may be limited by reduced adjacency. |
| Flagging all mines without revealing all cells | Not a win — all non-mine cells must also be revealed. |
