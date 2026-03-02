# Spatiotemporal Pricing and Fleet Management of Autonomous Mobility-on-Demand Networks: A Decomposition and Dynamic Programming Approach With Bounded Optimality Gap
## Summary
This paper develops a macroscopic network-flow model for joint spatiotemporal pricing and fleet management of autonomous mobility-on-demand (AMoD) systems that explicitly models demand elasticity to both price and waiting time. It proposes a relaxation, dual decomposition, and dynamic programming scheme to compute near-optimal control policies and derives a surrogate upper bound to quantify the optimality gap; the method is validated in a Manhattan case study using a microscopic simulator and MPC receding-horizon implementation.

## State
The environment is a discretized city represented as a graph of K zones with exogenous OD demand and fixed shortest-path trip times τij. System state tracks passenger and vehicle aggregate quantities: waiting passengers Qw, matched passengers Qm, in-vehicle passengers Qb (split into intra/inter-zone), idle vehicles Nv, relocating vehicles Nr, and parked (off-duty) vehicles Np. Continuous-time network-flow dynamics are specified (arrival, confirmation/matching, cancellation, pickup, trip completion, rebalancing) and discretized for computation; some relaxations aggregate inter-zone trips and treat Nv as a decision variable to decouple zones.

### Agent Role
The paper models a central platform/operator (fleet-level agent). Control variables, objective, and constraints are all defined at the platform level (origin-dependent price rates, rebalancing flows, and fleet size adjustments) and the objective maximizes platform profit; while a microscopic simulator is used for validation, individual drivers are not modeled as autonomous decision-makers. Therefore the agent is a company/fleet operator, not individual drivers.

## Action
The platform (central agent) controls: origin-dependent price rates pi(t), rebalancing flows rij(t) and fleet size adjustments si(t) in the original model. In the relaxed/decomposed formulation actions are effectively pi(t) and targeted number of idle/on-duty vehicles Nv_i(t). No machine-learning model is used; dynamic programming (value-function computation over a discretized smaller state space) and model predictive control are the computational tools.

## Transition
Environment updates follow the paper's network-flow differential equations (Eqs. (9)) that encode arrivals qi j(t)=q¯ij e^{−ϵ(α wi+pi τij)}, matching mij, cancellations m˜i, pickups bij (Cobb–Douglas form), and linear trip completion q˜ij ∝ Qbij/τij. The authors propose a relaxed transition model (Eqs. (13)) that aggregates inter-zone trips and assumes instantaneous repositioning for decomposition; this relaxed model is used to compute an upper bound. For validation and performance evaluation they run a microscopic agent-based simulator (20 s timestep) as the plant while using the macroscopic model as the MPC prediction model.

## Reward
Objective is platform profit over the horizon: integral over time of total fare revenue minus operational cost, i.e., ∫(Σ_{i,j} m_{ij}(t) p_i(t) τ_{ij} − γ N_on(t)) dt, subject to physical and operational constraints (price bounds, lower bound on idle vehicles, fleet size/parking limits, nonnegativity). This is formulated as a constrained optimal control problem (nonconvex); the relaxed problem provides an upper bound on this reward. Not an RL setup, so no policy/value-learning objective beyond dynamic-programming-based optimization.
