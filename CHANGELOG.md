# Changelog

All notable changes to this project are documented in this file.

## [0.9.0] - 2026-11-09

### Added

- Interactive 2D grid and multi-agent editor with six deterministic scenario
  families: `empty_room`, `random_obstacles`, `warehouse`, `maze`, `bottleneck`
  and `corner_swap`.
- Three comparable approaches on the same problem instance:
  - Prioritized Planning with Space-Time A* (`prioritized`).
  - Conflict-Based Search (`cbs`).
  - Multi-Agent PPO (`mappo`) with a pre-trained policy and optional
    on-instance training.
- Synchronized three-panel playback with shared timeline scrubbing, stepping,
  speed control and per-algorithm execution logs.
- Formal solution validation covering bounds/obstacles, legal movement, start
  and goal consistency, vertex conflicts, edge swaps and goal occupancy.
- Multi-seed batch experiments with success, validity, cost, makespan,
  suboptimality and timing summaries, plus CSV export.
- Browser playground that runs the real Python solvers through Pyodide, with a
  JavaScript fallback for offline use.
- Test suite covering the grid, heuristics, low-level search, prioritized
  planning, CBS, the validation pipeline, the RL environment and the research
  harness.

### Notes

- This repository adapts the open-source **PyMAPF** library by Erwin Lejeune
  and Sampreet Sarkar. The original library and its citation are preserved in
  `LICENSE`, `CITATION.cff` and `REFERENCES.md`.
