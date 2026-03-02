# Competitive pricing for ride-sourcing platforms with MARL
## Summary
This paper proposes a multi-agent reinforcement learning (MADDPG) approach integrated with a mesoscopic agent-based simulator to learn competitive spatial-temporal pricing between a human-driven vehicle (HV) platform and a shared autonomous vehicle (SAV) platform. Key findings show SAVs can obtain higher profit with fewer vehicles (no driver wages) and that spatial-temporal pricing yields higher profit than uniform dynamic pricing.

## State
The environment is a mesoscopic simulation over a real road network (Hangzhou) partitioned into 56 hexagonal regions. The joint state st = [sht, sat], where each platform state sht/sat comprises region-wise vectors: waiting passengers (dw), parked vehicles (vp), occupied vehicle destinations (vo), and sampled real-world demand distribution (dt). The setting is a partially observable MDP: each platform can observe its own state and the competitors’ observable actions (prices/wages) but not the competitor’s internal state.

### Agent Role
The paper models both a platform/company (the HV and SAV platforms) and drivers. The platforms are profit-maximizing companies that choose prices and wages; drivers are represented via participation functions and repositioning probabilities—captured at the individual-choice level but implemented as aggregate/steady-state flows rather than explicit per-driver controlled agents.

## Action
The joint action at = [aht, aat]. HV action aht = [wht, pht]: wht is the payout (driver wage) ratio and pht is a continuous price vector (price per km per region). SAV action aat = pat: a continuous region-wise price vector. ML model: MADDPG (multi-agent deep deterministic policy gradient) with actor-critic agents. Actors take the agent’s private state (and observe other platforms’ actions in execution) and output continuous price/wage vectors; critics are centralized during training and take full states and actions as inputs to evaluate Q-values.

## Transition
The environment updates are handled by the authors’ mesoscopic simulator (their own transition model, not an external MDP tool). Demand is sampled from real order data; passenger mode/platform choice follows a logit model using utilities (price, distance, SAV trust); matching is performed every 30s using the Kuhn–Munkres algorithm; routing uses a multi-level Dijkstra; vehicle state transitions (cruising, parking, occupied, offline) follow simulator rules. HV available fleet is modeled as a function of payout ratio (linear relation assumed); SAVs remain available except at night. The simulator therefore provides the transition dynamics used for training and experiments.

## Reward
Reward rt = [rht, rat] are per-time-step platform profits. HV profit rht = sum_i,sum_c p_ht_i * (1 - wht) * l_i,c^h (revenue after driver payout). SAV profit rat = sum_i,sum_c (p_at_i - DP - EC) * l_i,c^a (price minus depreciation and electricity costs). The RL objective is to maximize cumulative discounted profit; critics minimize TD error (Bellman loss) and actors are updated by policy gradients to maximize critic Q-values.


# Hybrid operations of human driving vehicles and automated vehicles with data-driven agent-based simulation
## Summary
This paper proposes D2ABMS, a data-driven agent-based modeling and simulation system to analyze hybrid operations of human-driven vehicles (HVs) and automated vehicles (AVs) in a large-scale urban ride-hailing market. Key contributions are a multi-objective deep learning driver decision model with learned embeddings to capture driver heterogeneity and a continuous-time, parallelized Go-based ABMS that integrates KM matching and OSRM routing to evaluate service, fleet, and environmental impacts.

## State
The environment is a real-world large-scale urban road network (Hangzhou, ~193 km2) with empirical demand (~130k trips/day) and OSM-derived network data. Agents: passengers (generate ride requests with origin, destination and wait tolerance), human drivers (HVs: online/offline state, working shifts, cruising/parking) and automated vehicles (AVs: park/cruise/pickup/deliver). Modeling assumptions include: maximum passenger waiting tolerance 30 min, boarding time 40 s, static shortest-path routing via OSRM, one passenger per vehicle, EPA-based emission factors, and baseline fleet sizes (e.g., 7000 HVs, 1540 AVs). Human driver heterogeneity is modeled by k-means clustering into 60 classes, with per-class embeddings learned by the neural network.

### Agent Role
Individual drivers and vehicles are modeled as per-agent entities: human drivers and AVs have per-agent states and decisions (e.g., offline decision, rest time, accept/decline). The platform acts as a centralized matching engine but is not modeled as an economic decision-making agent in the same way as platform-optimization papers.

## Action
Passengers: submit ride requests and wait (or cancel after tolerance). Vehicles (AVs/HVs): accept dispatches, drive to pickup, deliver passengers, cruise or park when idle, and reposition through cruise behavior. Human drivers additionally decide whether to go offline and the rest duration after going offline. ML model: a multi-objective deep neural network jointly predicts (1) IsOffline (binary classification) and (2) RestTime (regression). Inputs include per-order and driver features (e.g., current driving time, order price, last online hour, weekday, today's first online time, current working time, today/order counts and incomes) plus a learned embedding for the driver's class (60 classes). The network is trained end-to-end; the loss combines cross-entropy for the offline decision and mean squared error (MSE) for rest-time, weighted by a hyperparameter w. Matching/assignment uses the Kuhn–Munkres (Hungarian) algorithm with a utility that prioritizes longer-waiting passengers and nearer vehicles. Routing uses OSRM (fastest path).

## Transition
The environment updates in continuous time slices. At each slice the platform: collects agent states, solves a bipartite matching between waiting passengers and vacant vehicles (KM algorithm) based on a utility favoring waiting time and short pickup distance, dispatches vehicles, and advances vehicles along OSRM-derived routes according to time-slice length and vehicle speed. Vacant AVs park when idle; vacant HVs cruise with a set probability (e.g., 50%) and periodically decide to park. The simulation implements its own transition engine (D2ABMS) in Go and leverages existing tools for routing (OSRM) and matching; it does not propose a new microscopic traffic flow model but applies these components to update agent states and system-level variables.

## Reward
Reinforcement learning is not used. Objectives are expressed as agent and system performance metrics: passengers seek low waiting time and successful service (cancel after 30 min); vehicles/owners seek profit (revenue from time + distance fees, adjusted by peak/long-distance rules, minus fuel and parking costs); the platform evaluates average waiting time, order success rate, total VKT (deliver/pickup/cruise), fleet profit, and emissions (CO, HC, NOx estimated via EPA emission factors per VKT). The supervised driver model is trained to minimize a combined loss: cross-entropy (offline decision) + w * MSE (rest-time prediction). The assignment utility used in matching is a weighted function that increases with passenger waiting time and decreases with pickup distance (i.e., prioritizes closer vehicles and longer-waiting passengers).


# Learning-based control of AMoD in competitive environments
## Summary
This paper formulates multi-operator Autonomous Mobility-on-Demand (AMoD) control as a multi-agent reinforcement learning problem and shows that learning-based controllers (soft actor-critic with GNNs) can jointly learn rebalancing and dynamic pricing policies in competitive markets. It demonstrates robustness to added stochasticity from competitors, empirical convergence to equilibria, and analyzes effects of fleet size and entry of new competitors using a New York scenario.

## State
The environment is an urban-area partitioned into nodes (areas); each node's state captures local supply/demand statistics (vehicle counts, demand arrivals, trip lengths, etc.) and spatial relations are represented via a graph. Demand is simulated as Poisson arrivals with rates based on real-world data (Gammelli et al., 2022). Each operator (agent) centrally controls its fleet and does not observe competitors' internal states or actions.

### Agent Role
Agents correspond to company-level operators: each operator centrally controls its fleet (operator-level decision-makers). Individual drivers are not modeled as independent agents; control and rewards are defined at the operator/fleet level.

## Action
Agents output (a) a higher-level desired share of vehicles per area (modeled as a Dirichlet-parameterized action) which is converted by a minimum-cost rebalancing flow solver into concrete vehicle flows, and (b) a dynamic pricing control signal produced by the GNN actor (modeled as a Gaussian random variable used as a regression coefficient on travel time to determine trip prices). Both actor and critic are implemented with graph neural networks operating on area-level inputs.

## Transition
Environment updates by simulating Poisson trip arrivals, assigning trips probabilistically to operators (user choice proportional to a value-of-ride metric), applying cancellations modeled via a Hill sigmoid on price/length, moving vehicles according to the rebalancing flow solution, and updating vehicle locations/availability. The paper builds on and uses the simulation framework and New York scenario from Gammelli et al. (2022) rather than proposing a new transition model.

## Reward
Each agent's reward is the operator profit: revenue from accepted trips (price × accepted trips) minus rebalancing operational costs. The objective therefore balances served demand and pricing (affecting cancellations) against vehicle relocation costs; reported metrics include served demand, cancellations, rebalancing cost, average price, and total profit.


# Graph Meta-Reinforcement Learning for Transferable Autonomous Mobility-on-Demand
## Summary
This paper formulates multi-city AMoD control as a meta-reinforcement learning problem and proposes a recurrent graph-neural-network actor-critic that can rapidly adapt to unseen cities. Empirically, the meta-RL policy (RL2-style) attains near-oracle profits on held-out cities and is robust to distribution shifts like special events, congestion, and pricing changes.

## State
Each city is treated as a distinct MDP/task. The per-timestep state encodes the transportation graph (adjacency A) and node-level features X: current idle vehicles m_t,i, projected future idle availability {m_{t'}}, observed OD demand rates d_{t,ij}, prices p_{t,ij}, costs c_{t,ij}, plus previous action, reward, and done flag for recurrence. Assumptions: travel times and costs are exogenous and known; passenger arrivals follow inhomogeneous Poisson processes estimated from data; passengers leaving unmatched exit the system; AMoD fleet has negligible effect on network congestion (no endogenous congestion modeled). Tasks are sampled from a distribution p(T) over cities.

### Agent Role
The agent corresponds to the AMoD fleet operator (company-level). The model formulates operator-level actions and rewards (desired idle-vehicle distributions, operator profit) rather than per-driver decision-makers.

## Action
The agent outputs a desired idle-vehicle distribution a_t^{reb} over stations (a probability vector summing to 1 giving the percentage of idle vehicles to allocate to each node). The policy is parameterized by a recurrent graph neural network backbone (node encoder + GRU hidden states + graph pooling) within an Advantage Actor-Critic (A2C) meta-RL framework (RL2). The actor decoder maps graph node embeddings to Dirichlet concentration parameters alpha and samples a_t ~ Dirichlet(alpha). Inputs to the model are the graph structure and node features (X) together with recurrent hidden state that aggregates episode-level history.

## Transition
Environment dynamics are simulated rather than learned: passenger arrivals per OD are inhomogeneous Poisson processes (time-varying rates from real data), vehicle availability updates by conservation (incoming minus outgoing vehicles, accounting for passenger trips and rebalancing), and trip travel times/prices are treated as externally decided and known. The paper does not propose a new parametric transition model — it uses these assumptions and a simulator built from real taxi datasets; dispatching and conversion of desired distributions to rebalancing flows are solved via optimization (IBM CPLEX) as part of the three-step control loop (matching, RL rebalancing, post-processing minimal rebalancing cost).

## Reward
Reward at each timestep is operator profit: sum_{i,j} x_{t,ij} (p_{t,ij} - c_{t,ij}) - sum_{i!=j} y_{t,ij} c_{t,ij} (revenue from served passengers minus vehicle costs including rebalancing). The RL objective is to maximize expected discounted cumulative reward; meta-RL maximizes expected return over an entire trial (n episodes) sampled from p(T), encouraging fast adaptation. Implementation details: discount gamma=0.97, episode length T=20; training optimizes A2C returns (actor/critic pair) across meta-training tasks.


# Two-Sided Deep Reinforcement Learning for Dynamic Mobility-on-Demand Management with Mixed Autonomy
## Summary
This paper develops a two-sided multiagent deep reinforcement learning (DRL) framework to optimize real-time fleet management for mobility-on-demand (MoD) systems with mixed autonomy (human-driven conventional vehicles (CVs) and autonomous vehicles (AVs)). The platform (supervisor) centrally controls AV relocations and spatial–temporal commission fees to influence boundedly rational drivers, while drivers learn decentralized relocation policies; training uses a scalable multiagent advantage actor-critic (A2C) algorithm with a two-head policy network and mean-field approximations. Experiments on a NYC taxi simulator with real trip data show improved gross merchandise value, order fulfillment rate, and utilization versus benchmarks.

## State
The environment is a discretized urban service region (hexagonal grid) with time split into discrete intervals. Global state S for the supervisor includes AV distribution Va_t, CV distribution Vc_t (split by driver rationality types), aggregated OD trip requests D_t, and current time (one-hot). Drivers have partial observations o_m (grid index, time, and driver-type indicator). The problem is modeled as a two-sided multiagent MDP: a single supervisor agent (centralized) and many noncooperative driver agents (decentralized). Demand arrivals are stochastic and sampled from real NYC taxi data in the simulator.

### Agent Role
Both: a supervisor/platform agent (company-level) and many decentralized driver agents. The supervisor centrally controls AV relocations and commission parameters; drivers are modeled as decentralized agents learning relocation policies.

## Action
Supervisor agent actions: per-grid AV relocation decisions X_t (number of idle AVs to move to neighbor grids or stay) and spatial–temporal commission-fee parameters c_it (commission-rate coefficients) — implemented via a two-head policy network that outputs relocation probabilities and commission-rate distributions. Driver agent actions: discrete relocation choices (stay or move to a neighboring grid). ML used: multiagent A2C (policy and value networks); inputs to the supervisor policy network are the global state vector (Va, Vc, D, time), and inputs to driver policy are the local observation o_m. Drivers and supervisor share/train deep neural network actors and critics (drivers use mean-field approximation and share networks across homogeneous drivers).

## Transition
Environment updates are simulated: at each period orders are sampled (stochastic demand), a matching heuristic assigns passengers (priority to human drivers), matched vehicles enter service, and idle vehicles (AVs and CVs) relocate according to supervisor and driver actions; travel and pickup take one or more periods. The state transition is deterministic given actions and then randomized by newly sampled demand. The paper does not propose an analytical transition model for real networks but implements a detailed simulator (hex-grid, matching heuristic, sampled trip data) for training and evaluation — i.e., model-free (no closed-form P) and simulator-driven transitions.

## Reward
Supervisor objective: maximize discounted cumulative reward RA = sum γ^k r^A_k where immediate supervisor reward r^A_t = g^A_t + CF^A_t + ω f^A_t. Here g^A_t is platform trip profit, CF^A_t total commission fees collected, f^A_t number of fulfilled requests, and ω is a weight trading off profit vs. service (order fulfillment rate). Driver objective: each driver maximizes discounted cumulative compensation R^m = sum γ^k r^m_k where immediate reward r^m_t = g^m_t − CF^m_t − RC^m_t (trip profit minus commission fee minus relocation cost). Training uses A2C (actor–critic) with entropy regularization; drivers use mean-field Q approximation to handle many agents.


# SAMoD: Shared Autonomous Mobility-on-Demand using Decentralized Reinforcement Learning
## Summary
This paper proposes SAMoD, a fully decentralized multi-agent system where each vehicle learns relocation (rebalancing) and request-assignment (including dynamic ride-sharing) policies via reinforcement learning. Experiments on NYC taxi data using a simplified traffic simulator show SAMoD achieves near-centralized performance on served requests and waiting times while improving vehicle-oriented metrics (e.g., occupancy, empty VMT trade-offs).

## State
The environment is a free‑floating, one‑way shared mobility area discretized into zones; the authors use NYC taxi trip records for demand and a simplified grid-like travel-time model (no congestion, constant speed ~21 mph). Agents observe only local information: their vehicle state (empty / hasPassengers / full), presence of active requests in their current zone (yes/no) and in neighbouring zones (yes/no). Historical per-zone request counts are gathered online by agents for use in rebalancing decisions. The simulation assumes a fixed fleet (200 vehicles), vehicle capacity 4, and event-driven updates (zone border cross, drop-off, or rebalancing arrival).

### Agent Role
Individual vehicles are the agents: each vehicle (autonomous) learns per-vehicle relocation and pickup policies. There is no separate company/fleet operator modeled as a decision-making agent.

## Action
Each agent executes discrete actions A = {pickUp, rebalance, doNothing}. pickUp triggers nearest-request pickup (initial or ride‑sharing) and allows listening for further ride‑share requests while not full. rebalance selects a neighbouring zone according to one of learned strategies: (i) neighbour with most current pending requests, (ii) neighbour with largest supply-request gap, (iii) neighbour with historically most requests, or (iv) neighbour with historically largest gap. The learning method is tabular Q‑learning: Q(S,A) is updated per episode; state inputs are the discrete vehicle state and binary indicators of requests in current and neighbouring zones. Episodes are bounded by drop‑offs or end of rebalancing.

## Transition
Environment transitions are simulated: vehicles follow computed routes to pick‑ups, drop‑offs, or rebalance destinations; travel times are computed with a grid-based model and constant speed (no congestion modeling). Requests arrive from the historical NYC taxi dataset (used as demand trace). The paper does not propose a novel transition/dynamics model — it uses a simplified, event-driven simulator with service and movement dynamics and records historical per-zone statistics online.

## Reward
The RL objective is single‑signal and sparse: agents receive +100 reward when the vehicle has passenger(s) (i.e., successful pick‑up/serving) and 0 otherwise. Thus agents learn policies that maximize cumulative passenger-serving events; there are no explicit penalties for empty VMT or detours in the reward design (trade-offs emerge implicitly and are discussed; authors propose multi‑objective extensions for future work).


# Modeling the competition between multiple Automated Mobility on-Demand operators: An agent-based approach
## Summary
This paper develops an agent-based model (ABM) to study coexistence and competition between multiple Automated Mobility-on-Demand (AMoD) operators. It couples an endogenous multinomial logit demand model with operator fleet management (assignment and pricing) and a mesoscopic traffic simulator to quantify how fleet size, assignment algorithms and pricing strategies affect mode choice and key performance indicators (waiting time, travel time, served requests, empty VKT) in a case study of The Hague.

## State
The environment represents an urban AMoD morning-peak (5:30–10:00) in The Hague with 49 service centroids and ~25,800 travel requests. There are three competing AMoD operators, each managing a fleet of single-seat SAVs. The traffic environment is modeled mesoscopically (link/node movement rules, speed–density relationships). Agent types: individual travelers (requests), vehicles (operator fleets), fleet management centers (per operator), and a centralized traffic management center that provides routing and network state.

### Agent Role
Company/fleet operators and travelers: operators are modeled as company-level fleet managers making assignment and pricing decisions; travelers are individual demand-generating agents. Individual human drivers are not represented as independent decision-making agents.

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


# Competition and Cooperation of Autonomous Ridepooling Services: Game-Based Simulation of a Broker Concept
## Summary
This paper develops an agent-based simulation and turn-based game to evaluate how competition and broker-mediated cooperation among autonomous ridepooling providers affect profitability, pooling efficiency and customer service. Using a Manhattan taxi-data case study the authors compare four interaction scenarios (single operator, independent operators, broker with user choice, and regulated broker) and show that a regulated broker can nearly restore pooling efficiency and increase operator profit while slightly increasing waiting and detour times.

## State
Environment: discrete-time agent-based simulation on a street network G = (N, E) (OSMnx-extracted Manhattan); time step = 60 s; rebalancing and travel-time scaling every 15 min; demand based on subsampled NYC taxi trips (10% market penetration). Constraints: vehicle capacity cv = 4, maximum waiting time twait_max = 6 min, max relative in-vehicle detour Δ = 40%. Agents: customers (request tuples (i, ti, xsi, xdi)), operators (fleet controllers), vehicles (move on network, execute schedules), and a broker (in broker scenarios). Modeling: dynamic vehicle routing solved with insertion heuristics for offers and a global re-optimization via feasible vehicle-to-request-bundle (V2RB) enumeration + ILP; repositioning via Pavone-style rebalancing. Operators characterized by objective weights (cdis_α, cvot_α) and fleet size Nv. Interaction scenarios: Single Operator, Independent Operators (split demand), User Decision (broker forwards offers, user picks offer maximizing ϕ_user), Broker Decision (broker chooses offer maximizing ϕ_broker).

### Agent Role
Company/fleet operators (and a broker in broker scenarios): operators are the central decision-makers controlling offers, fleet schedules and repositioning. Vehicles and customers exist as assets and demand sources, but individual human-driver autonomy is not the focus (operators/broker control fleet behavior).

## Action
Customers: send requests to operator or broker and choose/book an offer when available (user utility ϕ_user depends on expected arrival/wait/travel time). Operators: create offers (user parameters ui,o — fare, expected waiting/travel times; system parameters si,o — additional driven distance), run local insertion heuristics to form offers, accept bookings, run global ILP assignment to assign V2RB schedules to vehicles, and periodically reposition idle vehicles. In the game, operators can change fleet size and objective weights (cdis_α, cvot_α) to maximize profit. Broker: forwards requests to operators and either presents offers to users (user decision) or selects offers to optimize a system utility ϕ_broker (minimize additional driven distance). No machine learning models are used; inputs are request tuples, current fleet state, travel times, and offer parameters; optimization uses heuristics and ILP.

## Transition
The environment advances in 60 s steps: new requests arrive, operators generate offers via insertion heuristics, booking decisions are resolved immediately, then a global re-optimization (ILP over V2RBs) updates vehicle schedules; vehicles and customers move along shortest-time paths; every Trepo = 15 min a repositioning (rebalancing) step adjusts vehicle distribution and travel times are scaled based on observed taxi trip travel times. Scope: case study for Manhattan using subsampled historical taxi data; transition dynamics are implemented by the paper’s own simulation framework (combining established algorithms: insertion heuristics, Alonso‑Mora-style V2RB+ILP, Pavone rebalancing, OSMnx for network). Not an RL environment — transitions are deterministic conditioned on stochastic demand draws and heuristics.

## Reward
No RL reward is used. Operators optimize economic profit: Profit P = R − C with R = sum_{i ∈ Cserved} ddirect_i · f (distance-dependent fare) and C = Nv · Cv + dfleet · cdis (fixed vehicle cost + distance-dependent operating cost). Effective profit maximized in the game is Peff = P − N_C,no · pno, where N_C,no counts requests that received no offer and pno is a penalty per unoffered request (calibrated). Operator scheduling objective (used to rank schedules) is ρ_α(ψ) = cdis_α · d(ψ) + cvot_α · Σ_i (t_arrival_i(ψ) − ti) − NR·|Rγ|, balancing fleet mileage and passenger value-of-time; broker system utility ϕ_broker(si,o) is measured by additional driven distance δd_i,o (broker selects offers that minimize system cost); user utility ϕ_user(ui,o) is modeled as minimizing customer arrival time (users pick fastest offered trip). Calibration values used in the study: Cv = 25 €/day, cdis = 0.25 €/km, initial cvot = 16.2 €/h, calibrated fare f ≈ 0.43 €/km and penalty pno ≈ 0.46 € per unoffered request.


# Spatiotemporal Pricing and Fleet Management of Autonomous Mobility-on-Demand Networks: A Decomposition and Dynamic Programming Approach With Bounded Optimality Gap
## Summary
This paper develops a macroscopic network-flow model for joint spatiotemporal pricing and fleet management of autonomous mobility-on-demand (AMoD) systems that explicitly models demand elasticity to both price and waiting time. It proposes a relaxation, dual decomposition, and dynamic programming scheme to compute near-optimal control policies and derives a surrogate upper bound to quantify the optimality gap; the method is validated in a Manhattan case study using a microscopic simulator and MPC receding-horizon implementation.

## State
The environment is a discretized city represented as a graph of K zones with exogenous OD demand and fixed shortest-path trip times τij. System state tracks passenger and vehicle aggregate quantities: waiting passengers Qw, matched passengers Qm, in-vehicle passengers Qb (split into intra/inter-zone), idle vehicles Nv, relocating vehicles Nr, and parked (off-duty) vehicles Np. Continuous-time network-flow dynamics are specified (arrival, confirmation/matching, cancellation, pickup, trip completion, rebalancing) and discretized for computation; some relaxations aggregate inter-zone trips and treat Nv as a decision variable to decouple zones.

### Agent Role
Company/platform-level agent: the platform is the central decision-maker controlling origin-dependent prices, rebalancing flows and fleet-size adjustments. Drivers are represented in aggregate quantities (participation/supply functions) and not as explicit per-driver agents.

## Action
The platform (central agent) controls: origin-dependent price rates pi(t), rebalancing flows rij(t) and fleet size adjustments si(t) in the original model. In the relaxed/decomposed formulation actions are effectively pi(t) and targeted number of idle/on-duty vehicles Nv_i(t). No machine-learning model is used; dynamic programming (value-function computation over a discretized smaller state space) and model predictive control are the computational tools.

## Transition
Environment updates follow the paper's network-flow differential equations (Eqs. (9)) that encode arrivals qi j(t)=q¯ij e^{−ϵ(α wi+pi τij)}, matching mij, cancellations m˜i, pickups bij (Cobb–Douglas form), and linear trip completion q˜ij ∝ Qbij/τij. The authors propose a relaxed transition model (Eqs. (13)) that aggregates inter-zone trips and assumes instantaneous repositioning for decomposition; this relaxed model is used to compute an upper bound. For validation and performance evaluation they run a microscopic agent-based simulator (20 s timestep) as the plant while using the macroscopic model as the MPC prediction model.

## Reward
Objective is platform profit over the horizon: integral over time of total fare revenue minus operational cost, i.e., ∫(Σ_{i,j} m_{ij}(t) p_i(t) τ_{ij} − γ N_on(t)) dt, subject to physical and operational constraints (price bounds, lower bound on idle vehicles, fleet size/parking limits, nonnegativity). This is formulated as a constrained optimal control problem (nonconvex); the relaxed problem provides an upper bound on this reward. Not an RL setup, so no policy/value-learning objective beyond dynamic-programming-based optimization.


# Geometric matching and spatial pricing in ride-sourcing markets
## Summary
This paper builds a spatial ride-sourcing model based on a discrete-time geometric matching process and uses it to study the effects of spatial (zonal) surge pricing and a simple regulation (a commission-rate cap). Main contributions: (1) identification of an inefficient "dominated" supply state (matching failures where many vehicle hours are spent en-route) and an analytic tipping condition for it; (2) characterization of platform optimal spatial pricing under short- and long-run labor supply and demonstration that revenue-maximizing spatial pricing can hurt consumers; (3) proposal and analysis of a commission-rate-per-unit-time cap that can recover a second-best under homogeneity assumptions.

## State
The environment is a city partitioned into homogeneous geographic zones. Time is modeled in discrete analysis periods subdivided by small time steps δ. Agents: customers (demand Qi per zone) and drivers (Ni vacant/occupied vehicles). Key modeling assumptions: matching is geometric and local — each requesting customer is matched to the closest vacant vehicle within radius r in the same zone; unmatched vehicle locations are approximated by a spatial Poisson point process; matching involves two delays (matching time until a match confirmation and en-route pickup time). Two labor-supply regimes are considered: perfectly elastic (long run, wage equals opportunity cost c) and perfectly inelastic (short run, fixed fleet size N). Demand is log-linear in fare and waiting time and is estimated from Didi order data.

### Agent Role
Both: the platform/operator and drivers. The platform is modeled as a profit-maximizing operator setting zonal prices and commission rates; drivers are represented via long-run/short-run labor-supply regimes—modeled at the individual-choice level but often implemented via aggregate supply functions or fixed N in the short run.

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


# Spatial pricing in ride-sourcing markets under a congestion charge
## Summary
The paper develops a network economic equilibrium model for ride-sourcing that links passenger demand, driver supply and repositioning, platform spatial pricing, waiting times, and congestion. It casts the platform's optimal spatial pricing under possible congestion charges as a large non-convex optimization, proposes an algorithm (grid search + dual decomposition) with a tight upper bound, and uses a San Francisco case study to compare one-directional cordon, bi-directional cordon and trip-based charges.

## State
The environment is a static, zonal representation of a city (M zones) connected by a road graph; zones are partitioned into a congested urban core C and a remote area R. Agents are passengers, drivers, a profit-maximizing platform, and a regulator that can impose congestion charges. Modeling assumptions include: long-run equilibrium (no fine temporal dynamics), passengers matched to nearest idle vehicle in the same zone, passenger and driver mode/participation choices follow logit-type choice models, passenger waiting time follows a decreasing function (square-root law in numerical study), driver intercept/repositioning probabilities are derived from an M/M/1 matching approximation, and congestion in C is captured via a speed function v_c(N_C) that feeds into trip times t_ij.

### Agent Role
Company/platform and drivers (equilibrium representation): the platform chooses location-differentiated fares and driver payments; drivers' participation and repositioning are modeled via logit/queueing-derived functions in a long-run equilibrium framework. The model focuses on equilibrium relations rather than individual agent control.

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
