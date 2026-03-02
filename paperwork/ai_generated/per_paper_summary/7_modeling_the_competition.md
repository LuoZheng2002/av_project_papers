# Modeling the competition between multiple Automated Mobility on-Demand operators: An agent-based approach
## Summary
This paper develops an agent-based model (ABM) to study coexistence and competition between multiple Automated Mobility-on-Demand (AMoD) operators. It couples an endogenous multinomial logit demand model with operator fleet management (assignment and pricing) and a mesoscopic traffic simulator to quantify how fleet size, assignment algorithms and pricing strategies affect mode choice and key performance indicators (waiting time, travel time, served requests, empty VKT) in a case study of The Hague.

## State
The environment represents an urban AMoD morning-peak (5:30–10:00) in The Hague with 49 service centroids and ~25,800 travel requests. There are three competing AMoD operators, each managing a fleet of single-seat SAVs. The traffic environment is modeled mesoscopically (link/node movement rules, speed–density relationships). Agent types: individual travelers (requests), vehicles (operator fleets), fleet management centers (per operator), and a centralized traffic management center that provides routing and network state.

## Action
Traveler agents: submit time-stamped OD requests and choose among the three operators according to a multinomial logit (MNL) utility. Operator agents: assign vehicles to requests (two tested methods: heuristic nearest-vehicle and an optimal bundle assignment solved via the Hungarian algorithm), set pricing (baseline, discount, or a supply–demand balancing rule), and dispatch vehicles along computed routes. Traffic manager: computes time-dependent shortest routes (Dijkstra) and updates link travel speeds.

If ML used: Not applicable — no machine learning or reinforcement learning methods are used. The demand model is an econometric discrete choice (MNL) whose inputs are operator-specific attributes: expected assignment time (queue/assignment delay), expected pickup time (from available idle vehicles), in-vehicle travel time (IVTT), and fare (base + distance + time, scaled by pricing factor). Utility is linear in these components.

## Transition
The environment advances in within-day simulation steps. When a request arrives it is (probabilistically) allocated among operators via the MNL based on current service attributes; the chosen operator runs an assignment algorithm to match idle vehicles to requests and issues a route. Vehicles transition between states (idle, en-route pickup, in-service, finished). The mesoscopic traffic simulator updates link densities and computes speeds via a speed–density relationship (piecewise function with critical density), which in turn affects travel and pickup times. Routing is computed at pickup and at trip start by a centralized traffic management center. The authors implement this mesoscopic traffic + routing integration within AnyLogic (i.e., they implement their own simulation integration rather than relying on an external traffic engine).

## Reward
Not applicable for reinforcement learning — no RL formulation is used. Traveler choice is driven by a utility function whose components are:
- V(wi): disutility of out-of-vehicle waiting (assignment time + expected pickup time), converted to monetary units using VOTT and a multiplier α;
- V(ti): disutility of in-vehicle travel time (IVTT × VOTT);
- V(fi): disutility of fare (fare × sensitivity parameters ε and η, with fare = base + distance×m + time×n).

Operators’ performance is evaluated by KPIs rather than an explicit optimization reward: served requests, average and quantile waiting times, average and quantile travel times, empty VKT, occupied VKT, total VKT and resulting congestion. Operators implicitly seek to attract demand and improve service-level KPIs (more served requests, lower waiting times and empty VKT), but no formal profit-maximization or RL reward function is provided.
