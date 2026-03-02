# Learning-based control of AMoD in competitive environments
## Summary
This paper formulates multi-operator Autonomous Mobility-on-Demand (AMoD) control as a multi-agent reinforcement learning problem and shows that learning-based controllers (soft actor-critic with GNNs) can jointly learn rebalancing and dynamic pricing policies in competitive markets. It demonstrates robustness to added stochasticity from competitors, empirical convergence to equilibria, and analyzes effects of fleet size and entry of new competitors using a New York scenario.

## State
The environment is an urban-area partitioned into nodes (areas); each node's state captures local supply/demand statistics (vehicle counts, demand arrivals, trip lengths, etc.) and spatial relations are represented via a graph. Demand is simulated as Poisson arrivals with rates based on real-world data (Gammelli et al., 2022). Each operator (agent) centrally controls its fleet and does not observe competitors' internal states or actions.

### Agent Role

The paper models each agent as a company/operator (fleet). Agents centrally control an entire fleet and optimize operator-level objectives (profit from pricing and rebalancing), rather than representing individual drivers. This is evidenced by fleet-level actions (rebalancing flows and dynamic pricing) and rewards defined as operator profit.

## Action
Agents output (a) a higher-level desired share of vehicles per area (modeled as a Dirichlet-parameterized action) which is converted by a minimum-cost rebalancing flow solver into concrete vehicle flows, and (b) a dynamic pricing control signal produced by the GNN actor (modeled as a Gaussian random variable used as a regression coefficient on travel time to determine trip prices). Both actor and critic are implemented with graph neural networks operating on area-level inputs.

## Transition
Environment updates by simulating Poisson trip arrivals, assigning trips probabilistically to operators (user choice proportional to a value-of-ride metric), applying cancellations modeled via a Hill sigmoid on price/length, moving vehicles according to the rebalancing flow solution, and updating vehicle locations/availability. The paper builds on and uses the simulation framework and New York scenario from Gammelli et al. (2022) rather than proposing a new transition model.

## Reward
Each agent's reward is the operator profit: revenue from accepted trips (price × accepted trips) minus rebalancing operational costs. The objective therefore balances served demand and pricing (affecting cancellations) against vehicle relocation costs; reported metrics include served demand, cancellations, rebalancing cost, average price, and total profit.
