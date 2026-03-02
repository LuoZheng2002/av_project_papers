# Graph Meta-Reinforcement Learning for Transferable Autonomous Mobility-on-Demand
## Summary
This paper formulates multi-city AMoD control as a meta-reinforcement learning problem and proposes a recurrent graph-neural-network actor-critic that can rapidly adapt to unseen cities. Empirically, the meta-RL policy (RL2-style) attains near-oracle profits on held-out cities and is robust to distribution shifts like special events, congestion, and pricing changes.

## State
Each city is treated as a distinct MDP/task. The per-timestep state encodes the transportation graph (adjacency A) and node-level features X: current idle vehicles m_t,i, projected future idle availability {m_{t'}}, observed OD demand rates d_{t,ij}, prices p_{t,ij}, costs c_{t,ij}, plus previous action, reward, and done flag for recurrence. Assumptions: travel times and costs are exogenous and known; passenger arrivals follow inhomogeneous Poisson processes estimated from data; passengers leaving unmatched exit the system; AMoD fleet has negligible effect on network congestion (no endogenous congestion modeled). Tasks are sampled from a distribution p(T) over cities.

### Agent Role
Company (fleet/operator): the agent is modeled as the AMoD fleet operator that controls rebalancing and dispatch decisions. Evidence: the action space specifies desired idle-vehicle distributions across stations, the reward is operator profit, and the simulator represents a central fleet rather than individual drivers. 

## Action
The agent outputs a desired idle-vehicle distribution a_t^{reb} over stations (a probability vector summing to 1 giving the percentage of idle vehicles to allocate to each node). The policy is parameterized by a recurrent graph neural network backbone (node encoder + GRU hidden states + graph pooling) within an Advantage Actor-Critic (A2C) meta-RL framework (RL2). The actor decoder maps graph node embeddings to Dirichlet concentration parameters alpha and samples a_t ~ Dirichlet(alpha). Inputs to the model are the graph structure and node features (X) together with recurrent hidden state that aggregates episode-level history.

## Transition
Environment dynamics are simulated rather than learned: passenger arrivals per OD are inhomogeneous Poisson processes (time-varying rates from real data), vehicle availability updates by conservation (incoming minus outgoing vehicles, accounting for passenger trips and rebalancing), and trip travel times/prices are treated as externally decided and known. The paper does not propose a new parametric transition model — it uses these assumptions and a simulator built from real taxi datasets; dispatching and conversion of desired distributions to rebalancing flows are solved via optimization (IBM CPLEX) as part of the three-step control loop (matching, RL rebalancing, post-processing minimal rebalancing cost).

## Reward
Reward at each timestep is operator profit: sum_{i,j} x_{t,ij} (p_{t,ij} - c_{t,ij}) - sum_{i!=j} y_{t,ij} c_{t,ij} (revenue from served passengers minus vehicle costs including rebalancing). The RL objective is to maximize expected discounted cumulative reward; meta-RL maximizes expected return over an entire trial (n episodes) sampled from p(T), encouraging fast adaptation. Implementation details: discount gamma=0.97, episode length T=20; training optimizes A2C returns (actor/critic pair) across meta-training tasks.
