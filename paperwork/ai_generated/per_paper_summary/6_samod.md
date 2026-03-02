# SAMoD: Shared Autonomous Mobility-on-Demand using Decentralized Reinforcement Learning
## Summary
This paper proposes SAMoD, a fully decentralized multi-agent system where each vehicle learns relocation (rebalancing) and request-assignment (including dynamic ride-sharing) policies via reinforcement learning. Experiments on NYC taxi data using a simplified traffic simulator show SAMoD achieves near-centralized performance on served requests and waiting times while improving vehicle-oriented metrics (e.g., occupancy, empty VMT trade-offs).

## State
The environment is a free‑floating, one‑way shared mobility area discretized into zones; the authors use NYC taxi trip records for demand and a simplified grid-like travel-time model (no congestion, constant speed ~21 mph). Agents observe only local information: their vehicle state (empty / hasPassengers / full), presence of active requests in their current zone (yes/no) and in neighbouring zones (yes/no). Historical per-zone request counts are gathered online by agents for use in rebalancing decisions. The simulation assumes a fixed fleet (200 vehicles), vehicle capacity 4, and event-driven updates (zone border cross, drop-off, or rebalancing arrival).

### Agent Role
The paper models each agent as an individual vehicle (an autonomous driver/vehicle). Agents correspond to vehicles in a fixed fleet of 200 and learn policies per vehicle; there is no separate company or fleet-operator decision-maker modeled. Thus: individual vehicle — yes; company/fleet/operator — no; both — no.

## Action
Each agent executes discrete actions A = {pickUp, rebalance, doNothing}. pickUp triggers nearest-request pickup (initial or ride‑sharing) and allows listening for further ride‑share requests while not full. rebalance selects a neighbouring zone according to one of learned strategies: (i) neighbour with most current pending requests, (ii) neighbour with largest supply-request gap, (iii) neighbour with historically most requests, or (iv) neighbour with historically largest gap. The learning method is tabular Q‑learning: Q(S,A) is updated per episode; state inputs are the discrete vehicle state and binary indicators of requests in current and neighbouring zones. Episodes are bounded by drop‑offs or end of rebalancing.

## Transition
Environment transitions are simulated: vehicles follow computed routes to pick‑ups, drop‑offs, or rebalance destinations; travel times are computed with a grid-based model and constant speed (no congestion modeling). Requests arrive from the historical NYC taxi dataset (used as demand trace). The paper does not propose a novel transition/dynamics model — it uses a simplified, event-driven simulator with service and movement dynamics and records historical per-zone statistics online.

## Reward
The RL objective is single‑signal and sparse: agents receive +100 reward when the vehicle has passenger(s) (i.e., successful pick‑up/serving) and 0 otherwise. Thus agents learn policies that maximize cumulative passenger-serving events; there are no explicit penalties for empty VMT or detours in the reward design (trade-offs emerge implicitly and are discussed; authors propose multi‑objective extensions for future work).
