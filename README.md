**Course:** Model Based Decisions (2025) — MSc Complex Systems & Policy
**Students:** Lars van den Heuvel(16433114) & Lena van Leeuwen(16421884)
**Instructor:** Michael Lees

---

## Objective of the Project 

This project simulates **Electric Vehicle (EV) adoption**, moddeled as a **coordination problem** using **Agent-Based Modelling (ABM)**. Individual agents choose one of two options: adopting an electronic vehicle (EV) i.e. Cooperate, (**C**) or stick to a conventional vehicle (CV) i.e. Defect (**D**). The interaction structure follows a **Stag Hunt–type game**, where EVs are attractive one enough others adopt to it.

One mechanism is the **feedback loop** between adoption and infrastructure:

* Higher EV adoption (**X**), the more charging infrastructure (**I**)
* The more infrastructure , the higher EV payoff (**a_I**)

We analyze how **initial conditions, network topology, and policy timing (subsidies)** affect tipping from low to high addoption 

---

## Model Overview

### Agents

* One agent per network node.
* Strategy: **C (EV)** or **D (CV)**.
* Payoffs depend on neighbors’ strategies and current infrastructure.
* Decisions are updated **synchronously** using either:

  * **Imitation** (copy highest-payoff neighbor), or
  * **Logit / softmax choice** with noise parameter (\tau).

### Infrastructure Dynamics

Infrastructure evolves endogenously:
[ I_{t+1} = I_t + g_I (X_t - I_t) ]
where (g_I) controls how fast infrastructure responds to adoption.

### Payoffs

* EV–EV interaction yields (a_I = a_0 + \beta_I I)
* EV–CV yields 0 for EV
* CV yields payoff (b), independent of neighbor

---

## Code Structure

### Core Classes

* **EVStagHuntModel (Mesa Model)**
  Manages the network, infrastructure dynamics, policy interventions, and synchronous scheduling.

* **EVAgent (Mesa Agent)**
  Computes payoffs in `step()` and commits decisions in `advance()` (SimultaneousActivation-safe).

### Strategy Selection

* `choose_strategy_imitate` — Agents copy the strategy of the highest-payoff neighbor.
* `choose_strategy_logit` — Agents choose probabilistically between EV and CV based on expected payoffs and noise parameter (tau).

### Analysis & Helper Functions

To support large-scale experiments and reproducibility, several helper functions are included:

* `run_network_trial` — Runs a single simulation until convergence and returns final adoption.
* `_row_for_ratio_task` — Internal helper for parallel computation of phase diagrams.
* `phase_sweep_X0_vs_ratio` — Computes heatmaps over initial adoption and payoff ratios.
* `run_phase_plot_X0_vs_I0_fixed_ratio` — Generates phase diagrams for initial adoption vs. infrastructure.
* `run_multi_X0_betaI_sweep` — Sensitivity analysis of infrastructure feedback strength (\beta_I).
* `cluster_formation` — Computes average cluster size of EV adopters in the network.
* `generate_network` — Creates random, small-world, or scale-free networks with fixed seeds.
* Plotting helpers (`plot_adoption_curves`, `plot_adoption_curves_all_networks_softcolors`) visualize adoption dynamics but do not affect model logic.

---

## Key Parameters used

| Parameter                          | What they entail                                   |
| ---------------------------------- | ---------------------------------------- |
| `X0_frac`                          | Initial fraction of EV adopters          |
| `a0`                               | Base EV payoff                           |
| `beta_I`                           | Infrastructure feedback strength         |
| `b`                                | ICE payoff                               |
| `g_I`                              | Infrastructure growth rate               |
| `I0`                               | Initial infrastructure level             |
| `tau`                              | Decision noise (logit model)             |
| `network_type`                     | `random`, `small_world`, or `scale_free` |
| `subsidy_a0_boost`                 | Temporary boost to (a_0) during policy   |
| `subsidy_start_t`, `subsidy_end_t` | Timing of subsidy                        |

---

## Experiments & Analysis

### Part 1 — Baseline Dynamics

* **Phase diagrams** of final adoption (X^*) over:

  * Initial adoption (X_0)
  * Payoff ratio (a_I / b)
* **Sensitivity analysis** of (\beta_I) to identify tipping regions.

### Part 2 — Network Structure

Comparison of:

* Random (Erdős–Rényi)
* Small-world (Watts–Strogatz)
* Scale-free (Barabási–Albert)

Metrics:

* Speed to high adoption
* Probability of reaching high-adoption equilibrium
* Mean cluster size of EV adopters
* Tipping sensitivity

### Part 3 — Policy Intervention

Evaluation of **EV subsidies modeled as temporary increases in (a_0)**:

* Baseline (no subsidy)
* Early subsidy (T1-T20)
* Mid subsidy (T30-T50)
* Late subsidy (T60-T80)
* Constant subsidy (T1-80)
(T portrays the timesteps in this case)

Each scenario is tested across all of the network types with independent seeds to ensure robustness.

---

## Outputs

The code produces:

* Heatmaps of final adoption equilibria
* Adoption curves (X(t)) across scenarios
* Network-comparative plots
* Summary statistics in pandas DataFrames

Plots are saved automatically to the `plots/` directory.

---

## How to Run

Run the full analysis from the main notebook listed below:

```
Assignment_3_MBDM.ipynb
```

### Requirements further specified in the requirements.txt

```
main packages:

pip install mesa networkx numpy pandas matplotlib
```

Python 3.10+ recommended.

---