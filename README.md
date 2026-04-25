# ⬡ Wumpus Logic Agent

A web-based Knowledge-Based Agent that navigates a Wumpus World-style grid using **Propositional Logic** and **Resolution Refutation** to deduce safe cells before moving. Features a retro gaming aesthetic with neon visuals and pixel-art typography.

---

## Overview

The agent operates in a randomly generated grid containing hidden Pits and a Wumpus. It has no prior knowledge of hazard locations — it must **sense**, **reason**, and **navigate** using a live Propositional Logic Knowledge Base (KB). Before entering any unvisited cell, the agent runs a full **Resolution Refutation** proof to confirm the cell is safe.

---

## Features

- **Dynamic Grid Sizing** — Configure any grid from 3×3 up to 8×8
- **Random Hazard Placement** — Pits and Wumpus are randomly placed each episode; configurable pit density (10%–40%)
- **Propositional Logic KB** — The agent maintains a CNF clause database that grows as it explores
- **Full Biconditional Encoding** — Both directions of `B ⟺ P1∨P2∨...` are encoded in CNF on every visit
- **Resolution Refutation Engine** — Proves `¬P_{r,c} ∧ ¬W_{r,c}` via contradiction before each move
- **Real-Time Metrics Dashboard** — Tracks inference steps, KB clause count, cells visited/proven safe
- **Full Grid Visualization** — Color-coded cells with agent position, percept indicators, and hazard reveals
- **Gaming UI** — Neon purple/green/gold color scheme with Press Start 2P pixel font and CRT scanline effect

---

## How It Works

### 1. Environment & Percepts

| Condition | Percept |
|-----------|---------|
| Adjacent to a Pit | 💨 Breeze |
| Adjacent to the Wumpus | 🟣 Stench |
| On the Gold cell | ✨ Glitter |

The agent starts at cell `(0,0)`, which is guaranteed safe. Axioms `¬P_0_0`, `¬W_0_0`, and `S_0_0` are added to the KB immediately.

### 2. Knowledge Base (TELL)

Each time the agent visits a cell `(r,c)`, it encodes the percepts as CNF clauses. Three facts about the current cell are told first:

```
¬P_{r,c}   — no pit here (agent is alive)
¬W_{r,c}   — no wumpus here (agent is alive)
S_{r,c}    — cell is safe
```

Then the full biconditional is encoded for both Breeze and Stench:

**Breeze biconditional** — `B_{r,c} ⟺ P_{adj1} ∨ P_{adj2} ∨ ...`

| Case | CNF Clauses Added |
|------|------------------|
| Breeze = true | `[B_{r,c}]` · `[¬B_{r,c} ∨ P_adj1 ∨ P_adj2 ∨ ...]` · `[¬P_adji ∨ B_{r,c}]` for each adj |
| Breeze = false | `[¬B_{r,c}]` · `[¬P_adj1]` · `[¬P_adj2]` · ... for each adj |

**Stench biconditional** — `ST_{r,c} ⟺ W_{adj1} ∨ W_{adj2} ∨ ...` — same encoding applied for Wumpus detection.

### 3. Inference Engine (ASK — Resolution Refutation)

Before moving to an unvisited cell `(r,c)`, the agent proves it is safe:

```
Goal:  ¬P_{r,c}  AND  ¬W_{r,c}

Method (per literal):
  1. Negate the goal: add [P_{r,c}] as a hypothesis clause to the KB copy
  2. Apply pairwise Resolution on all clause pairs
  3. If the empty clause ∅ is derived → CONTRADICTION → goal is PROVED
  4. Repeat for ¬W_{r,c}
  5. Only move if BOTH proofs succeed
```

Every resolution step is logged and displayed in the **Resolution Refutation Steps** panel. Each query is capped at 500 steps to prevent infinite loops.

### 4. Navigation Strategy

1. Check current cell for Gold (Glitter percept → episode won)
2. Run Resolution Refutation for each unvisited adjacent cell
3. Move to the first cell proven safe
4. If no adjacent safe cell exists, scan all globally proven-safe unvisited cells
5. If no safe frontier remains → agent reports **STUCK**

---

## Grid Cell States

| Color | Meaning |
|-------|---------|
| 🟡 Gold border (pulsing) | Agent's current position 🤖 |
| 🟢 Neon green | Proven safe, not yet visited |
| 🟣 Dark purple | Visited & safe |
| ⬛ Near-black | Unknown / unvisited |
| 🔴 Red | Confirmed Pit ⚫ |
| 💜 Hot pink | Confirmed Wumpus 👾 |
| ✨ Gold glow | Gold location 💎 |

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
2. Set **Grid Rows**, **Grid Cols**, and **Pit Density** using the left panel
3. Click **INITIALIZE WORLD** to generate a new episode
4. Use **STEP AGENT** to advance one move at a time, or **AUTO RUN** to let it run at 600ms intervals
5. Watch the KB Log and Resolution Steps panels for live reasoning output
6. Click **RESET** to reconfigure and start a new episode

---

## File Structure

```
wumpus_agent.html    — Single-file application (HTML + CSS + JS)
README.md            — This document
```

---

## Technical Notes

- **No external dependencies** — pure Vanilla JS, no frameworks or libraries required; fonts loaded from Google Fonts
- **CNF encoding** — all KB facts are stored as arrays of literals (disjunctive clauses); the full KB is their conjunction
- **Full biconditional CNF** — both the forward (`B → hazard`) and backward (`hazard → B`) directions are encoded, giving the resolution engine complete information
- **Visited-cell axioms** — on every move, `¬P_{r,c}` and `¬W_{r,c}` are added for the current cell, anchoring the proof chain
- **Tautology filtering** — resolvents containing both `P` and `¬P` are discarded automatically
- **Deduplication** — clause keys are canonicalized and stored in a `Set` to prevent redundant entries
- **Resolution cap** — each ASK query is capped at 500 resolution attempts to prevent infinite loops on undecidable queries
- **Grid orientation** — row `0` is displayed at the bottom of the grid (Wumpus World convention); `(0,0)` is bottom-left

---

## Bug Fixes (v1 → v2)

| Bug | Description | Fix |
|-----|-------------|-----|
| Missing `</style>` tag | CSS closing tag was dropped during reskin, causing all styles to render as raw page text | Restored `</style>` before `</head>` |
| Incomplete biconditional encoding | `tellBiconditional` only added the existential clause when sensor was active, missing the forward CNF clause `¬B∨P1∨P2∨...` and backward clauses `¬Pi∨B` | Rewrote to encode all required CNF directions |
| Duplicate safe-cell logic | `_updateKB` redundantly re-told safe-cell facts that conflicted with the corrected biconditional logic | Replaced with clean `¬P`/`¬W` unit-clause tells for visited cells |
| Unused `negateLit` function | Defined but never called anywhere in the codebase | Removed |
| Unused `clausesEqual` function | Defined but never called anywhere in the codebase | Removed |
| Unused `names` variable | Computed inside `resolve()` but never read | Removed |
| Unused `cont` variable | Return value of `agent.step()` assigned but never used | Removed assignment |

---

## Limitations

- The agent does not implement arrow/Wumpus-killing mechanics
- Pathfinding to a distant safe cell is direct (no A* or BFS); the agent teleports to any proven-safe unvisited cell when no adjacent safe move exists
- Resolution is complete for propositional logic but may not prove safety in all ambiguous configurations — the agent will report STUCK rather than guess

---

## Example Run (4×4 grid)

```
AXIOM: TELL [¬P_0_0]
AXIOM: TELL [¬W_0_0]
AXIOM: TELL [S_0_0]
PERCEPT at (0,0): Breeze=false, Stench=false
TELL [¬B_0_0]   — sensor inactive
TELL [¬P_1_0]   — no breeze → pit-free
TELL [¬P_0_1]   — no breeze → pit-free
TELL [¬ST_0_0]  — sensor inactive
TELL [¬W_1_0]   — no stench → wumpus-free
TELL [¬W_0_1]   — no stench → wumpus-free
ASK: Is (1,0) safe?
  QUERY: PROVE ¬P_1_0
  NEGATED HYPOTHESIS ADDED: [P_1_0]
  RESOLVE on P_1_0: [P_1_0] ⊕ [¬P_1_0] → [∅]
  → CONTRADICTION FOUND after 1 step
  QUERY: PROVE ¬W_1_0
  → CONTRADICTION FOUND after 1 step
PROVED: (1,0) is safe (2 steps)
MOVE → (1,0)
PERCEPT at (1,0): Breeze=true, Stench=false
TELL [B_1_0]
TELL [¬B_1_0 ∨ P_0_0 ∨ P_2_0 ∨ P_1_1]  — fwd biconditional
TELL [¬P_0_0 ∨ B_1_0]                   — bwd biconditional
TELL [¬P_2_0 ∨ B_1_0]                   — bwd biconditional
TELL [¬P_1_1 ∨ B_1_0]                   — bwd biconditional
```
