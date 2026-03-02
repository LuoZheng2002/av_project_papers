# MDP-oriented comparative summary (organized by: State, Action, Transition, Reward)

This file reorganizes the paper summaries by MDP components (State, Action, Transition, Reward). Papers with similar setups or modeling choices are grouped together under each component to highlight common approaches and differences.

---

## State (grouped by representation & observability)

- Graph / zone/node-level aggregate states (AMoD macroscopic / mesoscopic):
  - "Learning-based control of AMoD in competitive environments" — node-level features: vehicle counts, demand arrivals, trip lengths; spatial relations via a graph; each operator centrally controls its fleet and does not observe competitors' internal states.
  - "Graph Meta-Reinforcement Learning for Transferable AMoD" — adjacency A and node features X (idle vehicles, projected availability, OD demand rates, prices, costs); includes previous action/reward for recurrence across tasks (meta-RL).
  - "Spatiotemporal Pricing and Fleet Management (decomposition)" — aggregate zone-level quantities (waiting/matched/in-vehicle/idle/relocating vehicles); exogenous OD and fixed travel times.
  - "Spatial pricing under congestion charge" — static zonal equilibrium state (zone-level passenger/driver counts, congestion core vs remote, travel times tied to congestion function).

- Mesoscopic / region-partitioned simulation states (meso agent-based):
  - "Competitive pricing for ride-sourcing platforms with MARL" — region-wise vectors: waiting passengers, parked vehicles, occupied destinations, sampled demand distribution (56 hex regions, partial observability per platform).
  - "Hybrid operations (D2ABMS)" and "Modeling competition between AMoD operators (The Hague)" — mesoscopic/agent-based states with centroids/regions, vehicle/agent attributes, empirical demand traces, and network-derived travel times.

- Vehicle-centric local / decentralized states (fully decentralized SAMoD / per-vehicle):
  - "SAMoD" — each vehicle observes local zone state (vehicle state: empty/hasPassengers/full; presence of requests locally and in neighboring zones; short historical counts).

- Two-sided / hierarchical state (supervisor vs drivers):
  - "Two-Sided DRL for MoD with Mixed Autonomy" — global supervisor state: AV and CV distributions, aggregated OD requests, time; driver local partial observations (grid index, time, driver-type).

- Static / analytical equilibrium states (economic models):
  - "Geometric matching and spatial pricing", "Spatial pricing under congestion charge" — zonal aggregated quantities and equilibrium relations rather than time-stepped states; long-run (steady-state) vs short-run distinctions.

Notes: common assumptions include discretized spatial partitions (zones/hex/nodes), use of empirical demand traces or Poisson arrivals, and either partial observability for decentralized agents or centralized aggregated states for platform-level controllers.

---

## Action (grouped by action type & control granularity)

- Pricing & economic controls (continuous prices / commission rates):
  - Competitive pricing (HV vs SAV) — continuous region-wise price vectors and wage ratios (MADDPG actors output continuous prices/wages).
  - Spatiotemporal pricing (decomposition) — platform controls origin-dependent price rates pi(t) (continuous) and fleet-size/rebalancing targets in relaxed formulations.
  - Spatial pricing & congestion-charge papers — zonal fare rates r_i and driver payments/commission q; regulator actions set surcharge matrices.

- Rebalancing / relocation as distributional actions (Dirichlet / desired shares / flows):
  - "Learning-based control of AMoD" — desired share of vehicles per area (Dirichlet-parameterized action) converted to flows via a rebalancing solver.
  - "Graph Meta-RL" and many AMoD papers — desired idle-vehicle distributions or rebalancing flows (often converted by optimization solvers).
  - Two-Sided DRL supervisor — discrete per-grid AV relocation decisions (numbers to neighbor grids or stay).

- Vehicle-level discrete navigation / operational actions:
  - SAMoD: discrete actions {pickUp, rebalance, doNothing} per vehicle.
  - Hybrid D2ABMS / agent-based papers: vehicle actions include accept dispatch, drive to pickup, deliver, cruise, park, reposition; human drivers also choose offline decisions.

- Assignment / matching policy choices (heuristic/optimal assignment as agent action or environment mechanism):
  - Several papers separate matching (KM/Hungarian or insertion heuristics) as part of the environment transition; some treat pricing/rebalancing as actions while matching is an internal optimizer.

Notes: actions range from continuous pricing vectors and distributional rebalancing signals (platform-level continuous controls) to discrete per-vehicle maneuvers. Many learning methods output high-level targets that are post-processed by optimizers (flow solvers, matching algorithms).

---

## Transition (grouped by simulator vs analytical dynamics)

- Simulator-driven transitions (mesoscopic / microscopic ABMS or event-driven simulators):
  - Competitive pricing (mesoscopic simulator): demand sampled from real data, KM matching every 30s, routing with Dijkstra, vehicle state transitions via simulator logic.
  - Hybrid D2ABMS (Go-based continuous-time ABMS): matching via KM, routing via OSRM, continuous-time updates with dispatch and movement.
  - Two-Sided DRL, SAMoD, AMoD competition (The Hague), Competition & Cooperation (brokered pooling) — all implement their own simulators (discrete/continuous time, matching heuristics or ILP) to advance states.

- Stochastic arrival processes with optimization post-processing:
  - Learning-based AMoD, Graph Meta-RL: Poisson or inhomogeneous Poisson trip arrivals; rebalancing decisions converted to flows via optimization solvers; cancellations modeled probabilistically (price/length-dependent sigmoids).

- Analytical / aggregated dynamic models (network-flow, geometric matching, equilibrium):
  - Spatiotemporal Pricing (decomposition): differential equations / network-flow dynamics discretized for DP and MPC prediction model; relaxed instantaneous repositioning approximations used for decomposition.
  - Geometric matching: discrete-time geometric matching process with analytical approximations for matching and en-route pickup delays.
  - Spatial pricing under congestion charge: static equilibrium solved via algebraic constraints rather than time-stepped simulation.

Notes: many learning papers rely on an internal simulator to generate transitions (i.e., model-free training). Others use analytical transition models to enable optimization (DP/MPC) or theoretical equilibrium analysis.

---

## Reward / Objective (grouped by economic profit, service metrics, or supervised losses)

- Platform/operator profit (primary reward in RL papers):
  - Competitive pricing (HV/SAV), Learning-based AMoD, Graph Meta-RL, Two-Sided DRL (supervisor) — rewards are operator/platform profit: fares minus costs (driver wages, depreciation, electricity, rebalancing costs). Critics/actors optimize cumulative discounted profit.

- Multi-objective / composite platform + service metrics:
  - Two-Sided DRL explicitly composes supervisor reward from platform profit, commission fees, and order fulfillment (profit + CF + ω·fulfilled_requests).
  - Hybrid D2ABMS: no RL reward; evaluation metrics include waiting time, order success rate, fleet profit, emissions (objectives not learned but measured). The supervised driver model optimizes cross-entropy + weighted MSE.

- Agent-level rewards for decentralized learning:
  - SAMoD: sparse +100 when vehicle has passenger(s); agents maximize cumulative served events.
  - Driver agents in Two-Sided DRL: per-driver compensation objective (trip profit minus commission minus relocation cost).

- No-RL objectives (analytical/economic papers):
  - Geometric matching, Spatial pricing under congestion charge, Competition & Cooperation (broker) — central objectives are revenue/profit maximization or welfare optimization solved via optimization/analytical equilibrium; no RL reward function.

Notes: most RL-focused papers adopt operator profit as the reward signal, sometimes augmented with service-level terms. Decentralized vehicle agents may use sparse, service-centric rewards. Non-RL studies optimize economic objectives or welfare and use the models for equilibrium analysis or MPC.

---

## Grouped methodological patterns (how states/actions/transitions/rewards are combined)

- Centralized platform control with simulator transitions + operator profit reward:
  - e.g., Competitive pricing (MADDPG), Learning-based AMoD (soft actor-critic + GNN), Graph Meta-RL — common pattern: aggregated state, continuous pricing/rebalancing actions, simulator (or Poisson arrivals) for transitions, profit reward.

- Decentralized per-vehicle RL with local state and sparse service reward:
  - SAMoD: local observations, discrete actions, tabular Q-learning or decentralized policy learning, event-driven simulator for transitions, binary reward for serving passengers.

- Two-sided hierarchical control (supervisor + many drivers):
  - Two-Sided DRL: supervisor learns high-level controls (AV relocations, commission rates) with global state and platform reward; drivers are modeled with decentralized learning or behavioral models (mean-field approximations) with their own objectives.

- Analytical economic models used for theory / bounds / MPC:
  - Spatiotemporal Pricing decomposition and Geometric matching: use aggregated dynamic or equilibrium models to derive optimal pricing policies or characterize inefficiencies and bounds; these are tools for planning/optimization rather than RL policy learning.

---

References (by short title as in the original file):
- Competitive pricing for ride-sourcing platforms with MARL
- Hybrid operations of human driving vehicles and automated vehicles with data-driven agent-based simulation (D2ABMS)
- Learning-based control of AMoD in competitive environments
- Graph Meta-Reinforcement Learning for Transferable AMoD
- Two-Sided Deep Reinforcement Learning for Dynamic MoD Management with Mixed Autonomy
- SAMoD: Shared Autonomous Mobility-on-Demand using Decentralized Reinforcement Learning
- Modeling the competition between multiple Automated Mobility on-Demand operators: An agent-based approach
- Competition and Cooperation of Autonomous Ridepooling Services: Game-Based Simulation of a Broker Concept
- Spatiotemporal Pricing and Fleet Management (decomposition)
- Geometric matching and spatial pricing in ride-sourcing markets
- Spatial pricing in ride-sourcing markets under a congestion charge

---

If you want, I can:
- further merge entries to produce a compact comparison table (zones, action types, transition model, reward), or
- generate per-paper cross-references showing which papers share the same state/action/transition/reward choices, or
- produce a shorter 1-page cheat-sheet prioritized by RL vs non-RL distinctions.
