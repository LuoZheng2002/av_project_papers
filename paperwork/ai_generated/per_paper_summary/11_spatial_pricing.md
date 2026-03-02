# Spatial pricing in ride-sourcing markets under a congestion charge
## Summary
The paper develops a network economic equilibrium model for ride-sourcing that links passenger demand, driver supply and repositioning, platform spatial pricing, waiting times, and congestion. It casts the platform's optimal spatial pricing under possible congestion charges as a large non-convex optimization, proposes an algorithm (grid search + dual decomposition) with a tight upper bound, and uses a San Francisco case study to compare one-directional cordon, bi-directional cordon and trip-based charges.

## State
The environment is a static, zonal representation of a city (M zones) connected by a road graph; zones are partitioned into a congested urban core C and a remote area R. Agents are passengers, drivers, a profit-maximizing platform, and a regulator that can impose congestion charges. Modeling assumptions include: long-run equilibrium (no fine temporal dynamics), passengers matched to nearest idle vehicle in the same zone, passenger and driver mode/participation choices follow logit-type choice models, passenger waiting time follows a decreasing function (square-root law in numerical study), driver intercept/repositioning probabilities are derived from an M/M/1 matching approximation, and congestion in C is captured via a speed function v_c(N_C) that feeds into trip times t_ij.

## Action
Platform: chooses location-differentiated per-time fares r_i and driver payment (or commission) q to maximize profit subject to equilibrium constraints.

Passengers: choose mode (take ride-sourcing or not) by comparing generalized cost c_ij = alpha * w^p_i + beta * t_ij + r_i * t_ij; demand is lambda_ij = lambda0_ij * F_p(c_ij) (logit form used in experiments).

Drivers: decide long-run participation N = N0 * F_d(q) and short-run repositioning (stay or cruise to zone j) with logit probabilities P_ij based on expected per-time earnings and waiting/travel times; idle drivers may be intercepted while repositioning (intercept probability derived from per-zone match probability sigma_i).

Regulator: sets congestion charge structure (one-directional cordon, bi-directional cordon, or trip-based), represented by per-trip or per-crossing surcharge matrix pbar_ij that is passed to passengers or drivers depending on the scheme.

Machine learning: Not applicable — the paper does not use ML. Instead it uses economic choice models (logit) and analytical queueing approximations (M/M/1).

## Transition
The environment is updated to a steady-state (equilibrium) via a system of algebraic constraints rather than a dynamic simulator: vehicle-hour conservation (partition of N into occupied, pickup, idle hours), network flow balance (inflow = outflow in each zone), pickup/matching and intercept probabilities (from M/M/1), and the congestion speed function v_c(N_C) that determines travel times t_ij. The model is a static spatial equilibrium (no continuous-time transition dynamics); the paper proposes this equilibrium/transition model (it does not rely on an external traffic simulator). Numerical experiments solve the equilibrium for given policy parameters (platform choices and congestion charges) over a San Francisco zonal dataset.

## Reward
Not applicable to reinforcement learning. The central optimization objective is the platform's profit: total platform revenue (sum over i,j of r_i * t_ij * lambda_ij) minus total driver payment (N * q) and, when applicable, minus congestion charges borne by drivers/platform (sum of ftilde_ij * indicator_d_ij * pbar_ij). The optimization is constrained by the demand model, waiting-time bounds, driver supply (N = N0 F_d(q)), vehicle-hour conservation, congestion speed dependence on N_C, and network flow balance. The paper also reports passenger surplus, driver surplus and tax revenue as welfare metrics when evaluating congestion-charge schemes.
