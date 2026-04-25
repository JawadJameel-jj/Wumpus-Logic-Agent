# ⬡ Wumpus Logic Agent

A web-based Knowledge-Based Agent that navigates a Wumpus World-style grid using **Propositional Logic** and **Resolution Refutation** to deduce safe cells before moving.

---

## Overview

The agent operates in a randomly generated grid containing hidden Pits and a Wumpus. It has no prior knowledge of hazard locations — it must **sense**, **reason**, and **navigate** using a live Propositional Logic Knowledge Base (KB). Before entering any unvisited cell, the agent runs a full **Resolution Refutation** proof to confirm the cell is safe.

---

## Features

- **Dynamic Grid Sizing** — Configure any grid from 3×3 up to 8×8
- **Random Hazard Placement** — Pits and Wumpus are randomly placed each episode; configurable pit density (10%–40%)
- **Propositional Logic KB** — The agent maintains a CNF clause database that grows as it explores
- **Resolution Refutation Engine** — Proves `¬P_{r,c} ∧ ¬W_{r,c}` via contradiction before each move
- **Real-Time Metrics Dashboard** — Tracks inference steps, KB clause count, cells visited/proven safe
- **Full Grid Visualization** — Color-coded cells with agent position, percept indicators, and hazard reveals

---

## How It Works

### 1. Environment & Percepts

| Condition | Percept |
|-----------|---------|
| Adjacent to a Pit | 💨 Breeze |
| Adjacent to the Wumpus | 🟣 Stench |
| On the Gold cell | ✨ Glitter |

The agent starts at cell `(0,0)`, which is guaranteed safe.

### 2. Knowledge Base (TELL)

Each time the agent visits a cell `(r,c)`, it encodes the percepts as CNF clauses:

**Breeze biconditional** — `B_{r,c} ⟺ P_{adj1} ∨ P_{adj2} ∨ ...`

Encoded in CNF as:
- If breeze: `[P_{adj1} ∨ P_{adj2} ∨ ...]` (at least one neighbor has a pit)
- If no breeze: `[¬P_{adj1}]`, `[¬P_{adj2}]`, ... (all neighbors are pit-free)

**Stench biconditional** — `ST_{r,c} ⟺ W_{adj1} ∨ W_{adj2} ∨ ...`

Same encoding applied for Wumpus detection.

### 3. Inference Engine (ASK — Resolution Refutation)

Before moving to an unvisited cell `(r,c)`, the agent proves it is safe:

```
Goal:  ¬P_{r,c}  AND  ¬W_{r,c}

Method (per literal):
  1. Negate the goal: add [P_{r,c}] as a hypothesis clause to the KB
  2. Apply pairwise Resolution on all clause pairs
  3. If the empty clause ∅ is derived → CONTRADICTION → goal is PROVED
  4. Repeat for ¬W_{r,c}
  5. Only move if BOTH are proved
```

Every resolution step is logged and displayed in the **Resolution Refutation Steps** panel.

### 4. Navigation Strategy

1. Check current cell for Gold (Glitter → episode won)
2. Query KB for each unvisited adjacent cell
3. Move to the first cell proven safe
4. If no adjacent safe cell exists, scan all globally proven-safe unvisited cells
5. If no safe frontier remains → agent reports **STUCK**

---

## Grid Cell States

| Color | Meaning |
|-------|---------|
| 🟡 Gold border (pulsing) | Agent's current position |
| 🟢 Green | Proven safe (not yet visited) |
| 🔵 Dark blue | Visited & safe |
| ⬛ Dark gray | Unknown / unvisited |
| 🔴 Red | Confirmed Pit |
| 🟣 Purple | Confirmed Wumpus |

Hazards are **only revealed** on the grid when the agent dies or wins (episode over).

---

## Metrics Dashboard

| Metric | Description |
|--------|-------------|
| Total Inference Steps | Cumulative resolution steps across all ASK queries |
| Agent Moves | Number of cells the agent has moved to |
| KB Clauses | Current number of CNF clauses in the Knowledge Base |
| Cells Visited | Cells the agent has physically entered |
| Cells Proven Safe | Cells formally proved safe via resolution |
| Agent Position | Current `(row, col)` coordinates |
| Episode Status | `ACTIVE` / `DEAD` / `WON` / `STUCK` |

---

## Usage

1. Open `wumpus_agent.html` in any modern browser — no build step, no dependencies
2. Set **Grid Rows**, **Grid Cols**, and **Pit Density**
3. Click **INITIALIZE WORLD** to generate a new episode
4. Use **STEP AGENT** to advance one move at a time, or **AUTO RUN** to let it run continuously
5. Watch the KB Log and Resolution Steps panels for live reasoning output
6. Click **RESET** to configure and start a new episode

---

## File Structure

```
wumpus_agent.html    — Single-file application (HTML + CSS + JS)
README.md            — This document
```

---

## Technical Notes

- **No external dependencies** — pure Vanilla JS, no frameworks or libraries required
- **CNF encoding** — all KB facts are stored as arrays of literals (disjunctive clauses); the full KB is a conjunction of these
- **Tautology filtering** — resolvents containing both `P` and `¬P` are discarded automatically
- **Deduplication** — clause keys are canonicalized and stored in a `Set` to prevent redundant entries
- **Resolution cap** — each ASK query is capped at 500 resolution attempts to prevent infinite loops on undecidable queries
- **Grid orientation** — row `0` is displayed at the bottom of the grid (Wumpus World convention); `(0,0)` is bottom-left

---

## Limitations

- The agent does not implement arrow/Wumpus-killing mechanics
- Backtracking through already-visited cells to reach a safe frontier is approximated (not full path planning)
- Resolution is complete for propositional logic but may not prove safety in all ambiguous configurations — the agent will report STUCK rather than guess

---

## Example Run (4×4 grid)

```
TELL [¬P_0_0]         — AXIOM: No pit at (0,0)
TELL [¬W_0_0]         — AXIOM: No wumpus at (0,0)
PERCEPT at (0,0): B=false, S=false
TELL [¬P_1_0]         — No breeze → (1,0) is pit-safe
TELL [¬P_0_1]         — No breeze → (0,1) is pit-safe
ASK: Is (1,0) safe?
  QUERY: PROVE ¬P_1_0
  → CONTRADICTION FOUND (∅) after 3 steps
  QUERY: PROVE ¬W_1_0
  → CONTRADICTION FOUND (∅) after 5 steps
PROVED: (1,0) is safe (8 steps)
MOVE → (1,0)
PERCEPT at (1,0): B=true, S=false
TELL [P_0_0 ∨ P_2_0 ∨ P_1_1] — Breeze → ∃ pit in neighbors
```
