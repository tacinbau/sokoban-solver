# Sokoban Solver — room decomposition + push-optimal BFS

[![make.fun · sokoban-solver-01-run-20261121](https://make.fun/badge/sokoban-solver-01-run-20261121.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)
[![make.fun · sokoban-solver-01-run-20261121 · rank](https://make.fun/badge/sokoban-solver-01-run-20261121/tacinbau/sokoban-solver/rank.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)
[![make.fun · sokoban-solver-01-run-20261121 · reward](https://make.fun/badge/sokoban-solver-01-run-20261121/tacinbau/sokoban-solver/reward.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)

[Build the best Sokoban solver, 21 November 2026](https://make.fun/bounty/sokoban-solver-01-run-20261121)

A dependency-free Node 22 program (`main.ts`, builtins only) that solves
every level of a set in the push-optimal number of pushes.

## How it works

A generated level is a 1-wide hallway with 1–4 small rooms off it, and a
box pushed out of a room's door into the hallway can never be pushed back —
so every box stays in its own room and the level's push-optimum is the
**sum of the rooms' push-optima**. A whole-level search over 16 boxes is
infeasible; a per-room search is tiny. So `main.ts`:

1. **Splits the level into rooms** — cuts the hallway's dead squares out of
   the floor and takes the connected components that hold boxes.
2. **Solves each room with a push-optimal BFS** over pushes (state = sorted
   box cells + the player's reachable region, named by its lowest cell;
   first solved state found is push-optimal), pruned by **dead squares**
   (floor cells from which no goal is reachable, found by reverse flood
   from the goals) and **frozen 2×2 blocks** of walls/boxes.
3. **Stitches the rooms together** with BFS hallway walks; other rooms'
   boxes stand as walls while a room is solved (on their goals once done,
   so the player is never shut out). Every solution is **verified by
   replay** before it is written; a whole-level push search is the
   fallback if a level does not decompose.

The search is budgeted by **expansion count, never by the clock** — no
`Date`, no `Math.random`, no environment reads — so two runs write
byte-identical output.

## Run locally

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=<kit>/practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
MAKEFUN_OUTPUT_DIR=out MAKEFUN_REFERENCE_DIR=<kit>/practice/reference \
  MAKEFUN_SCORES_DIR=scores node <kit>/program/scorer/score.ts
```

## Practice-set score

**1,000,000 / 1,000,000** — all 5 instances solved in P\* pushes
(13, 16, 29, 47, 60), in ~1 s for the whole set. The house baseline
plain-astar 1.0.0 scores 200,000 on the same set (only i001 solved).
Six further sets drawn fresh from the kit generator with different seeds
also all scored 1,000,000, each in under 1 s.
