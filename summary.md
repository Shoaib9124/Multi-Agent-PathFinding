# 📑 PathWeave: Multi-Agent Pathfinding (MAPF) Research & Technical Summary

**Course / Domain**: Artificial Intelligence · Multi-Agent Systems · Robotics & Pathfinding  
**Platform**: PathWeave (v0.9.0)  
**Workspace**: `Multi-Agent-PathFinding`  
**Key Modules**: `pathweave.core`, `pathweave.algorithms`, `pathweave.rl`, `pathweave.research`  
**Validation Suite**: 162 unit & integration tests (`pytest`)  

---

## 1. Executive Summary (The Elevator Pitch)

**PathWeave** is a reproducible 2D Multi-Agent Pathfinding (MAPF) simulation, research, and benchmarking platform. Its purpose is to systematically evaluate and compare three distinct paradigms of multi-agent coordination on identical grid environments under strict formal physical invariants:

1. **Decoupled Heuristic Search**: Prioritized Space-Time A*
2. **Coupled Optimal Search**: Conflict-Based Search (CBS)
3. **Multi-Agent Reinforcement Learning (MARL)**: Multi-Agent Proximal Policy Optimization (MAPPO) with Centralized Training and Decentralized Execution (CTDE)

The platform provides an interactive 3-panel synchronized playback web application, an automated batch benchmarking engine exporting to CSV, a formal 6-rule solution validator, and a pure-NumPy neural network engine with analytical backpropagation.

---

## 2. What the Code Does (Key Capabilities)

* **2D Grid MAPF Problem Formulation**: Solves discrete pathfinding where $N$ agents navigate from assigned start cells to unique goal cells using 4-connected cardinal moves (`UP`, `DOWN`, `LEFT`, `RIGHT`) or `WAIT`.
* **6 Benchmark Scenario Presets**:
  * `empty_room`: Open space testing crossing traffic.
  * `random_obstacles`: Maze-like obstacle field testing detour routing.
  * `warehouse`: Parallel rack corridors testing structured directional flow.
  * `maze`: Single-path topologies testing dead-end avoidance.
  * `bottleneck`: 1-cell chokepoints between rooms testing queuing and capacity limits.
  * `corner_swap`: Symmetrical head-on position swaps testing deadlock resolution.
* **Formal Solution Validator (`validate_solution`)**: Audits every proposed trajectory against 6 strict invariants:
  1. *In-bounds & Obstacle-Free*: Every visited cell is within grid bounds and free of walls.
  2. *Legal 4-Connected Moves*: Every step is either a cardinal step ($\|v^{t+1} - v^t\|_1 \le 1$) or a wait action.
  3. *Vertex Conflict-Free*: No two agents occupy the same cell at time $t$ ($v_i^t \neq v_j^t$).
  4. *Edge-Swap Conflict-Free*: No two agents cross the same undirected edge simultaneously ($v_i^t = v_j^{t+1} \land v_j^t = v_i^{t+1}$).
  5. *All Agents Reach Goal*: Every agent terminates on its assigned goal.
  6. *Stay-at-Goal Occupancy (One-Shot MAPF)*: Agents remain parked on their goal cell after arrival and block traffic for remaining agents up to the makespan.
* **Synchronized 3-Panel Playback UI**:
  * An interactive browser application (`http://localhost:3000`) with a grid editor, simultaneous scrubbing across all 3 algorithms on a unified discrete timeline ($t = 0 \dots H_{\max}$), and real-time execution logs.
  * Dual-execution engine: Runs directly against the Python backend or client-side inside the browser via Pyodide / WebAssembly fallback.
* **Standardized Metric Logging**:
  * **Sum of Costs ($SIC$)**: $\sum_{i=1}^N (\text{len}(p_i) - 1)$.
  * **Makespan ($MK$)**: $\max_i (\text{len}(p_i) - 1)$.
  * **True Suboptimality Ratio**: $SIC_{\text{algo}} / SIC^*_{\text{CBS}}$ evaluated against the provably optimal CBS baseline.
  * **Separation of Times**: Explicitly isolates *online search planning time* from *offline RL training time* and *online RL inference time*.

---

## 3. How It Does It (Technical Architecture)

```
                            ┌─────────────────────────────────┐
                            │    MAPFProblem (Grid + Agents)  │
                            └────────────────┬────────────────┘
                                             │
         ┌───────────────────────────────────┼───────────────────────────────────┐
         ▼                                   ▼                                   ▼
┌──────────────────┐               ┌──────────────────┐               ┌──────────────────┐
│   Prioritized    │               │  Conflict-Based  │               │      MAPPO       │
│  Space-Time A*   │               │   Search (CBS)   │               │     (MARL)       │
├──────────────────┤               ├──────────────────┤               ├──────────────────┤
│ Sequential order │               │ Two-level search │               │ Actor: 9x9 local │
│ Space-time table │               │ Constraint Tree  │               │ Critic: Global   │
│ Polynomial time  │               │ Optimal for SIC  │               │ O(1) step infer. │
└────────┬─────────┘               └────────┬─────────┘               └────────┬─────────┘
         │                                   │                                   │
         └───────────────────────────────────┼───────────────────────────────────┘
                                             ▼
                                  ┌─────────────────────┐
                                  │   validate_solution │
                                  │   (6 MAPF Invariants)│
                                  └──────────┬──────────┘
                                             ▼
                                  ┌─────────────────────┐
                                  │ Comparative Metrics │
                                  │ & Synchronized Play │
                                  └─────────────────────┘
```

### 3.1 Prioritized Planning (`pathweave/algorithms/prioritized_planning.py`)
* Agents plan sequentially according to a priority order ($a_1 \succ a_2 \succ \dots \succ a_N$).
* Higher-priority paths populate a **space-time reservation table** with vertex reservations $(v, t)$ and edge reservations $(u, v, t)$.
* Lower-priority agents search around prior paths using **Space-Time A\*** where states are 3D coordinates $((r, c), t)$.
* **Complexity**: $O(N \cdot |V| \cdot H)$ — fast ($<10\text{ ms}$), but **incomplete** (vulnerable to livelocks and deadlocks on adversarial priority orderings).

### 3.2 Conflict-Based Search (`pathweave/algorithms/cbs.py`)
* **Two-Level Search**:
  * **High-Level**: Best-first search over a binary **Constraint Tree (CT)**. Each node stores agent constraints, individual paths, sum-of-costs, and conflict count (used as the tie-breaker).
  * **Low-Level**: Runs Space-Time A* only for the single agent whose constraints changed.
* **Branching**: When the earliest conflict occurs between Agent $A$ and Agent $B$ at cell $C$ and timestep $t$, CBS forks:
  * *Child 1*: Forbid $(C, t)$ for Agent $A$.
  * *Child 2*: Forbid $(C, t)$ for Agent $B$.
* **Guarantee**: Provably **complete and optimal** for sum-of-costs.

### 3.3 Multi-Agent Reinforcement Learning (`pathweave/rl/`)
* **PettingZoo Parallel API Environment (`env.py`)**: Models simultaneous agent moves with cascading collision physics (refused actions iteratively cascade to a fixed point).
* **Observation Encoder (`observation.py`)**:
  * Egocentric $9\times 9$ window with 4 binary channels (walls, other agents, own goal, other goals).
  * Concatenated with normalized relative goal distance and bearing vectors ($327$ total features).
* **Centralized Training with Decentralized Execution (CTDE)**:
  * Actor policy $\pi_\theta(a_i | o_i)$ receives only local $327$-dim observations for decentralized deployment.
  * Centralized critic $V_\phi(s)$ receives the normalized global state ($5N+1$ features) during training to stabilize the value baseline and reduce variance.
* **Pure NumPy Neural Network Engine (`networks.py`)**:
  * Implements MLPs with explicit forward passes, analytical backpropagation, and the Adam optimizer in pure NumPy without heavy external ML dependencies.
  * Analytical gradients are numerically validated in tests against finite-difference calculus.
* **Potential-Based Reward Shaping (`reward.py`)**:
  * Potential function $\Phi(s)$ is computed via backward BFS true shortest-path distance on the grid: $R(s, s') = \gamma \Phi(s') - \Phi(s)$.
  * Guides exploration toward goals from timestep 0 without altering the optimal policy.

---

## 4. Algorithmic Comparison Matrix

| Dimension | Prioritized Space-Time A* | Conflict-Based Search (CBS) | MAPPO (MARL) |
| :--- | :--- | :--- | :--- |
| **Optimality Guarantee** | None (suboptimal) | **Guaranteed Optimal** ($SIC$) | None (learned heuristic) |
| **Completeness** | **Incomplete** (priority traps) | **Complete** (finds path if exists) | Bounded by horizon $H_{\max}$ |
| **Computational Class** | Polynomial $O(N \cdot \|V\| \cdot H)$ | Exponential $O(2^C \cdot \|V\| \cdot H)$ | $O(1)$ per step (Inference) |
| **Coordination Model** | Sequential reservation table | High-level constraint tree | Centralized critic / Local actor |
| **Communication at Test**| Global reservation access | Centralized search | None (purely local perception) |
| **Bottleneck Behavior** | Fails on opposing flows | Solves via tree branching | High collision / oscillation rate |

---

## 5. Necessary Details to Say to Your Professor

### 1. Research Scope & Trade-offs
> *"We investigated the trade-off between **classical centralized search** (complete and optimal, but computationally exponential) and **decentralized reinforcement learning** (instantaneous $O(1)$ step inference, but susceptible to local deadlocks without global perspective)."*

### 2. Fair Experimental Methodology (Search vs. Training Time)
> *"Comparing search time directly to RL training time is an apples-to-oranges mistake common in weak literature. PathWeave strictly decomposes timing into:
> 1. **Online search planning time** (Prioritized A*, CBS).
> 2. **Offline RL training / fine-tuning time**.
> 3. **Online RL rollout inference time** ($O(1)$ forward pass per agent per step)."*

### 3. True Suboptimality Baseline
> *"Rather than benchmarking against an arbitrary heuristic, both Prioritized Planning and MAPPO are benchmarked against CBS, which provides the **provably optimal ground truth** sum-of-costs ($SIC^*$). Suboptimality is evaluated as $SIC / SIC^*_{\text{CBS}}$."*

### 4. Stay-at-Goal Permanent Occupancy
> *"In realistic one-shot MAPF, agents do not disappear upon reaching their goal. They stay parked and block other agents. PathWeave enforces this: an agent cannot settle on its goal during low-level Space-Time A* if another agent has a constraint passing through that cell at a later timestep."*

### 5. Why Prioritized Planning Fails in Bottlenecks
> *"In scenarios like `bottleneck` or `corner_swap`, higher-priority agents reserve narrow passages in perpetuity. Lower-priority agents are starved of valid trajectories and return `None` (unsolvable), even though valid coordinated detour plans exist."*

### 6. Software Engineering Rigor
> *"The search algorithms have **zero external runtime dependencies** (pure standard library). The neural network engine is written in pure NumPy with hand-derived backpropagation verified against finite differences. The test suite comprises **162 passing tests**."*

---

## 6. Anticipated Professor Questions & Model Answers

### Q1: *"Why not just run standard A\* on the joint configuration space?"*
* **Answer**:
  > *"Joint-space A\* searches in $V^N$. For $N=10$ agents on a $20\times 20$ grid, the state space is $400^{10} \approx 10^{26}$, with a branching factor of $5^{10} \approx 9.7 \times 10^6$. CBS bypasses this explosion by decomposing the problem into $N$ single-agent searches and resolving conflicts lazily only when paths intersect."*

### Q2: *"What is the corridor symmetry problem in CBS?"*
* **Answer**:
  > *"When two agents attempt to cross a narrow hallway in opposite directions, standard CBS resolves a collision at cell $c$ and time $t$ by branching, which simply pushes the collision to $t+1$ or cell $c+1$. The Constraint Tree expands exponentially ($2^L$) exploring redundant permutations of who enters first. This is clearly observed in our `bottleneck` benchmark."*

### Q3: *"How does MAPPO address multi-agent non-stationarity?"*
* **Answer**:
  > *"In decentralized learning, the environment appears non-stationary because other agents change their policies concurrently. By utilizing a **centralized critic** that observes the joint state $\mathbf{s}_t \in \mathbb{R}^{5N+1}$ during training, the value baseline remains stationary, stabilizing policy gradients and enabling Generalized Advantage Estimation (GAE) to reduce variance."*

### Q4: *"How are collisions handled during RL rollout evaluation?"*
* **Answer**:
  > *"Rollouts do not allow ghosting or teleportation. Illegal moves and collisions are refused in simulation, with refusals cascading to a fixed point. The resulting trajectory is evaluated by the same formal validator as the search planners; if an agent fails to reach its goal or conflicts, the rollout is marked `Solved: False`."*

---

## 7. Five-Minute Presentation Script

* **Minute 1 (Context & Challenge)**: Introduce MAPF, its applications in automated warehouses and AGV fleets, and the exponential $|V|^N$ complexity of joint state space.
* **Minute 2 (The 3 Approaches)**: Outline Prioritized Space-Time A* (decoupled heuristic), CBS (optimal two-level search), and MAPPO (decentralized CTDE learning).
* **Minute 3 (Architecture & Invariants)**: Explain the 6 formal MAPF invariants, the stay-at-goal occupancy rule, and the pure-NumPy neural network engine with finite-difference gradient checks.
* **Minute 4 (Empirical Insights)**: Show empirical results: CBS yields minimum cost but scales poorly in bottlenecks; Prioritized Planning is fast ($<10\text{ ms}$) but fails under opposing flows; MAPPO provides instantaneous inference but suffers from local perception blindness in symmetrical swaps.
* **Minute 5 (Demo & Conclusion)**: Demonstrate the CLI output (`python main.py --cli-demo`) or the interactive web platform on `localhost:3000` with synchronized 3-panel playback.

---

## 8. Verification & Quick Commands

```bash
# 1. Run headless CLI comparison
python main.py --cli-demo

# 2. Launch interactive web simulation platform (Port 3000)
python main.py

# 3. Run the automated test suite (162 tests)
python -m pytest -q
```

