# Competitive pricing for ride-sourcing platforms with MARL
## Summary
This paper proposes a multi-agent reinforcement learning (MADDPG) approach integrated with a mesoscopic agent-based simulator to learn competitive spatial-temporal pricing between a human-driven vehicle (HV) platform and a shared autonomous vehicle (SAV) platform. Key findings show SAVs can obtain higher profit with fewer vehicles (no driver wages) and that spatial-temporal pricing yields higher profit than uniform dynamic pricing.

## State
The environment is a mesoscopic simulation over a real road network (Hangzhou) partitioned into 56 hexagonal regions. The joint state st = [sht, sat], where each platform state sht/sat comprises region-wise vectors: waiting passengers (dw), parked vehicles (vp), occupied vehicle destinations (vo), and sampled real-world demand distribution (dt). The setting is a partially observable MDP: each platform can observe its own state and the competitors’ observable actions (prices/wages) but not the competitor’s internal state.

## Action
The joint action at = [aht, aat]. HV action aht = [wht, pht]: wht is the payout (driver wage) ratio and pht is a continuous price vector (price per km per region). SAV action aat = pat: a continuous region-wise price vector. ML model: MADDPG (multi-agent deep deterministic policy gradient) with actor-critic agents. Actors take the agent’s private state (and observe other platforms’ actions in execution) and output continuous price/wage vectors; critics are centralized during training and take full states and actions as inputs to evaluate Q-values.

## Transition
The environment updates are handled by the authors’ mesoscopic simulator (their own transition model, not an external MDP tool). Demand is sampled from real order data; passenger mode/platform choice follows a logit model using utilities (price, distance, SAV trust); matching is performed every 30s using the Kuhn–Munkres algorithm; routing uses a multi-level Dijkstra; vehicle state transitions (cruising, parking, occupied, offline) follow simulator rules. HV available fleet is modeled as a function of payout ratio (linear relation assumed); SAVs remain available except at night. The simulator therefore provides the transition dynamics used for training and experiments.

## Reward
Reward rt = [rht, rat] are per-time-step platform profits. HV profit rht = sum_i,sum_c p_ht_i * (1 - wht) * l_i,c^h (revenue after driver payout). SAV profit rat = sum_i,sum_c (p_at_i - DP - EC) * l_i,c^a (price minus depreciation and electricity costs). The RL objective is to maximize cumulative discounted profit; critics minimize TD error (Bellman loss) and actors are updated by policy gradients to maximize critic Q-values.
