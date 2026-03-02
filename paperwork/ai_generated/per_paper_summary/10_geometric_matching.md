# Geometric matching and spatial pricing in ride-sourcing markets
## Summary
This paper builds a spatial ride-sourcing model based on a discrete-time geometric matching process and uses it to study the effects of spatial (zonal) surge pricing and a simple regulation (a commission-rate cap). Main contributions: (1) identification of an inefficient "dominated" supply state (matching failures where many vehicle hours are spent en-route) and an analytic tipping condition for it; (2) characterization of platform optimal spatial pricing under short- and long-run labor supply and demonstration that revenue-maximizing spatial pricing can hurt consumers; (3) proposal and analysis of a commission-rate-per-unit-time cap that can recover a second-best under homogeneity assumptions.

## State
The environment is a city partitioned into homogeneous geographic zones. Time is modeled in discrete analysis periods subdivided by small time steps δ. Agents: customers (demand Qi per zone) and drivers (Ni vacant/occupied vehicles). Key modeling assumptions: matching is geometric and local — each requesting customer is matched to the closest vacant vehicle within radius r in the same zone; unmatched vehicle locations are approximated by a spatial Poisson point process; matching involves two delays (matching time until a match confirmation and en-route pickup time). Two labor-supply regimes are considered: perfectly elastic (long run, wage equals opportunity cost c) and perfectly inelastic (short run, fixed fleet size N). Demand is log-linear in fare and waiting time and is estimated from Didi order data.

## Action
Agents' actions and platform control:
- Customers: generate requests and accept service implicitly via the demand function (no explicit cancellation modeled once matched).
- Drivers: choose zone to operate to equalize expected payoff (long-run equal-profit condition) or are fixed in number (short run).
- Platform: sets zonal price multipliers (spatial pricing) and (in the long-run formulation) commission percentage to maximize revenue. Optimization problems are formulated for long-run (SP-L) and short-run (SP-S) revenue maximization; flat-pricing counterparts (FP-L/FP-S) are used for comparison. No machine-learning models are used.

## Transition
The environment updates in discrete time via the paper's discrete-time geometric matching process: at each small time step δ all unmatched/new requests are considered; the platform loops randomly through requesting customers and assigns the closest vacant vehicle within radius r if available; matched pairs are removed and then incur an en-route pickup time before the vehicle becomes occupied. The paper derives analytical/approximate expressions for expected matching time and en-route time as functions of unmatched-customer/vehicle intensities, zone area, r, speed v and a depletion parameter α. This geometric matching specification is proposed by the authors (i.e., their transition model) and is calibrated/used for equilibrium and welfare analysis (δ ≈ 12s, α ≈ 0.5, ψ for unmatched-intensity scaling) using Didi data.

## Reward
Not applicable as an RL problem. Instead, objectives are:
- Platform objective: revenue maximization (∑ zones η Fi Qi) subject to demand, matching waiting-time relations, vehicle conservation, and labor-supply constraints (SP-L and SP-S formulations). In SP-L the platform can choose commission η and zonal prices; in SP-S fleet is fixed and only prices are chosen.
- Welfare/objective benchmarks: consumer surplus CS defined as ∫_0^F f(z) dz − β w Q (per zone) and producers' surplus PS = Fi Qi − c Ni (joint platform+drivers normalized profit). The paper formulates a second-best social welfare problem (SB-L) under spatial equilibrium constraints and shows a commission-rate-per-unit-time cap can align platform incentives with maximizing occupied vehicle hours and — under homogeneity and technical assumptions (e.g., λi ≠ 1) — implement the second-best.
