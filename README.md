# Moral-Swarm-Ethical-Autonomous-Disaster-Intelligence-System
A fully explainable ethical AI system for coordinating drone swarms in disaster response with real-time replanning and auditable decisions.

A MATLAB research-quality prototype of an AI decision engine for an
autonomous disaster-response drone swarm. Built for clarity, modularity,
reproducibility, and explainability.

## How to run (Download the zip file "Moral Swarm_v")

1. Open MATLAB 
2. Open this folder as your working directory (or run `addpath` on it).
3. Run:
   ```matlab
   main
   ```
   This runs the entire pipeline end-to-end: scenario generation → sensor
   detection → ethical prioritization → drone assignment (optimization) →
   explainable AI reporting → simulated disruption + dynamic replanning →
   performance metrics → visualization dashboard.

4. Inspect the `mission` struct left in your workspace afterward for full
   results (`mission.victims`, `mission.drones`, `mission.assignments`,
   `mission.explanations`, `mission.metrics`, `mission.replanLog`).

## Toolbox requirements

- **Statistics and Machine Learning Toolbox** — `betarnd` (sensor detection
  confidence model in `detectVictims.m`).
- **Optimization Toolbox** — `intlinprog` (binary integer program for
  optimal drone-victim assignment in `assignDrones.m` / `optimizeAssignments.m`).
  If unavailable, the code automatically falls back to a greedy heuristic
  (see "Fallback behavior" below).
- No Deep Learning, Reinforcement Learning, or Parallel Computing Toolbox
  is required — the scoring/assignment model is intentionally a
  transparent weighted-sum + linear-assignment formulation, which is more
  auditable and explainable than a black-box learned model for this
  use case. (See `ethicalPriority.m` header for rationale.)

## File structure

| File                    | Purpose                                                            |
|--------------------------|---------------------------------------------------------------------|
| `main.m`                 | Top-level pipeline driver / orchestration script                  |
| `generateScenario.m`     | Procedurally generates victims, drones, hazards, obstacles         |
| `detectVictims.m`        | Simulates CV/thermal/environmental sensor fusion & detection noise |
| `ethicalPriority.m`      | Computes configurable weighted ethical priority score per victim   |
| `assignDrones.m`         | Solves the victim→drone assignment problem (ILP, with greedy fallback) |
| `optimizeAssignments.m`  | Standalone, reusable LAP solver (used by `assignDrones.m` and `updateMission.m`) |
| `explainDecision.m`      | Generates structured, human-readable explanations per decision     |
| `updateMission.m`        | Simulates disruptions (drone failure, new victim, low battery) and triggers replanning |
| `metrics.m`              | Computes 7 performance metrics (response time, fairness, etc.)     |
| `visualization.m`        | Renders the 7-panel visualization dashboard                        |

## Configuration

All tunable parameters live in the `config` struct at the top of `main.m`:
- Scenario size (number of victims/drones/hazards/etc., map dimensions)
- Ethical priority weights (must sum to ~1.0; a warning is issued otherwise)
- Assignment constraints (max distance, drone speed, battery reserve)
- Replanning triggers (toggle simulated drone failure / new victim discovery)

Change these values and re-run `main` to explore different scenarios.

## Fallback behavior

`assignDrones.m` and `optimizeAssignments.m` will use `intlinprog` for an
exact optimal assignment if Optimization Toolbox is licensed. If not
available (or if the solve fails for any reason), both functions
automatically fall back to a documented greedy nearest-cost heuristic, so
the pipeline still runs end-to-end without that toolbox — just with a
non-guaranteed-optimal (but still reasonable) assignment.

## Explainability

Every assignment decision produces a structured explanation
(`mission.explanations`), including:
- Decision, Assigned Drone, Priority Score, Confidence
- Reasoning (plain-language bullet list tied to underlying attributes)
- Alternative Choice (the runner-up drone and why it was not chosen)

This is intentionally a transparent rule-based/optimization model rather
than a trained black-box classifier, so every number in the explanation
traces directly back to an auditable formula.

## Reproducibility

`main.m` seeds the RNG (`rng(42, 'twister')`) at the start, so re-running
the script produces identical scenarios and decisions. Change the seed to
explore different randomized disasters.

## Known limitations / extension points

- `detectVictims.m` models sensor noise but does not yet read real CV/
  thermal hardware streams — replace its body with real sensor ingestion
  to use this in a live system.
- The decision-tree visualization is a schematic of the scoring formula
  (since the engine deliberately avoids a black-box trained tree, for
  auditability) rather than a fitted classification tree. If you add a
  learned classifier elsewhere in the pipeline, `fitctree`/`view` could
  generate a literal decision tree plot.
- Path planning treats drone-to-victim travel as straight-line distance;
  swap in an A*/RRT planner in `assignDrones.m`'s cost computation for
  obstacle-aware routing.
