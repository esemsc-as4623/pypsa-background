#### Stochastic dual dynamic programming (SDDP) algorithm for multistage stochastic mixed-integer non-linear optimization
- https://www.youtube.com/watch?v=danccm4dftY
- stochastic problem has stage-wise independent structures
	- at each stage there are possibilities
- transitional probabilities between stages are independent of scenarios in the precedent stage and current scenario
- dual dynamic programming solves inductively with forward and backward stage
	- forward: find solution at each stage
	- backward: use data at last stage to update approximation in precedent stage
	- converge asymptotically if convex
- achieve generalization of SDDP type algorithms to general (mixed-integer / non-convex) multistage problems
	- regularization for non-Lipschitz continuous value functions and conditions for exactness
	- generalized conjugacy for deriving valid inequalities for approximation updates
- unified framework for iteration complexity
- [summary]
	- solve sequential decision-making problems under uncertainty where the problem contains
		- multi-stage decisions
		- stochasticity (uncertainty) that evolve over stages
		- mixed-integer variables (continuous and discrete)
		- non-linearity of objective function and constraints
- [link]
	- PyPSA is designed for deterministic and two-stage stochastic optimization (using scenario-based modelling)
	- applying SDDP involves using PyPSA to formulate and solve the stage-wise subproblems within an outer SDDP framework

#### Reinforcement learning (RL) for integer programming (IP): learning to cut
- https://www.youtube.com/watch?v=1NjBIdcBB0w
- types of IP problems
	- branch-and-bound = solver finds an optimal solution by repeatedly splitting a node into two subproblems by choosing a fractional variable to split the feasible region, forming a search tree
	- cutting plane = solver iteratively adds linear constraints to tighten the feasible region and close the integrality gap (the difference between the optimal IP and linear program solutions)
	- greedy algorithms on graphs = make the locally optimal choice at each stage with the hope of finding a globally optimal solution
- ML to spin up or optimize certain aspects of IP
- Markov Decision Process (MDP)
	- state space, S, action space, A, policy maps from state to action
	- start with state $s_t$, sample action $a_t$ from policy, arrive at new state $s_{t+1}$ with an immediate reward $r_t$
	- maximize cumulative expected return over time
	- weighted sum over rewards
	- expectation over the policy; objective is a function of the policy
- [summary]
- [link]
	- PyPSA optimization is generally large-scale MILP
	- RL agent can intervene in the internal decisions of the MILP
		- e.g. unit commitment binary on/off status
		- e.g. investment capacity (binary variable for component, continuous variable for capacity)

#### An analysis of branch-and-bound for random multidimensional knapsack
- https://www.youtube.com/watch?v=OAsBtE8kIV0
- multidimensional knapsack problem (MKP) is a combinatorial optimization
	- set of items with value and weights
- [link]
	- frame the specific energy optimization subproblem as a knapsack-type problem
	- capacity expansion investment decisions
		- select the optimal set of infrastructure investments (items) to maximize system benefits or minimize total cost subject to constraints (dimensions)
	- resource allocation for demand-side management
		- select a portfolio of demand-side actions to maximize profit under limits on available flexibility resources

A unified approach to mixed-integer optimization: non-linear formulations and scalable algorithms

#incomplete 