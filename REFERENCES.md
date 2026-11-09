# References

The algorithms implemented in this repository and the work they are based on.

## Multi-agent pathfinding

| Work | Reference |
|---|---|
| MAPF definitions, variants and benchmarks | Stern, R.; Sturtevant, N. R.; Felner, A.; Koenig, S.; Ma, H.; Walker, T. T.; Li, J.; Atzmon, D.; Cohen, L.; Kumar, T. K. S.; Boyarski, E.; and Barták, R. 2019. *Multi-Agent Pathfinding: Definitions, Variants, and Benchmarks.* SOCS 2019: 151–158. |
| Survey behind the PyMAPF library | Lejeune, E.; and Sarkar, S. 2021. *Survey of the Multi-Agent Pathfinding Solutions.* DOI 10.13140/RG.2.2.14030.28486. |

## Single-agent search

| Algorithm | Module | Reference |
|---|---|---|
| Dijkstra | `pymapf.algorithms.search.dijkstra` | Dijkstra, E. W. 1959. *A note on two problems in connexion with graphs.* Numerische Mathematik 1(1): 269–271. |
| A* | `pymapf.algorithms.search.astar` | Hart, P. E.; Nilsson, N. J.; and Raphael, B. 1968. *A formal basis for the heuristic determination of minimum cost paths.* IEEE Transactions on Systems Science and Cybernetics 4(2): 100–107. |
| Space-Time A* | `pymapf.algorithms.space_time_astar` | Silver, D. 2005. *Cooperative pathfinding.* AIIDE 2005: 117–122. |

## Multi-agent planners

| Algorithm | Module | Reference |
|---|---|---|
| Prioritized Planning | `pymapf.algorithms.prioritized_planning` | Silver, D. 2005. *Cooperative pathfinding.* AIIDE 2005: 117–122. |
| Conflict-Based Search (CBS) | `pymapf.algorithms.cbs` | Sharon, G.; Stern, R.; Felner, A.; and Sturtevant, N. R. 2015. *Conflict-based search for optimal multi-agent pathfinding.* Artificial Intelligence 219: 40–66. |

## Learning-based planning

| Algorithm | Module | Reference |
|---|---|---|
| Proximal Policy Optimization (PPO) | `pymapf.rl.algorithms` | Schulman, J.; Wolski, F.; Dhariwal, P.; Radford, A.; and Klimov, O. 2017. *Proximal Policy Optimization Algorithms.* arXiv:1707.06347. |
| Generalized Advantage Estimation (GAE) | `pymapf.rl.algorithms` | Schulman, J.; Moritz, P.; Levine, S.; Jordan, M.; and Abbeel, P. 2016. *High-Dimensional Continuous Control Using Generalized Advantage Estimation.* ICLR 2016. |
| Multi-Agent PPO (MAPPO) | `pymapf.rl.algorithms` | Yu, C.; Velu, A.; Vinitsky, E.; Gao, J.; Wang, Y.; Bayen, A.; and Wu, Y. 2022. *The Surprising Effectiveness of PPO in Cooperative Multi-Agent Games.* NeurIPS 2022. arXiv:2103.01955. |

## Implementation notes

- Movement is four-connected (up, down, left, right) plus a wait action.
  Diagonal movement is disabled by default.
- The reported sum of costs counts movement and waiting up to each agent's goal
  arrival time. Padding added only to synchronize playback is not counted.
- In one-shot MAPF, an agent remains on its goal after arrival and blocks later
  arrivals; the validator checks this occupancy rule explicitly.
- The MAPPO policy uses a radius-4 local observation window with a goal vector
  and a centralized critic during training.
