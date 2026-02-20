a response to the perfect foresight issue

ESM investment decisions are sequential, adaptive, uncertain, path-dependent

- sequential decision-making under uncertainty
	- state = installed capacity, demand, fuel price, carbon price, technology cost, storage levels, grid constraints, climate conditions
	- actions = investment and operational decisions (build, retire, invest, expand, implement demand response, etc.)
	- transition model = represent demand evolution, fuel price dynamics, technology learning curves, policy shifts, weather patterns
	- reward = social welfare - system cost - emissions damage
	- discount factor = represent time preference, intergenerational equity, social discount rate
	- output = contingent strategies (e.g. if gas prices rise, build renewables faster; if demand stagnates, delay nuclear; if storage cost drops, accelerate electrification)
- current role of MDP in operations, not long-term planning
	- operational control (e.g. battery storage control where agent learns when to charge / discharge)
	- power system operation (e.g. economic dispatch, unit commitment, load scheduling where RL learns policies balancing cost and reliability)
	- microgrid energy management (learn optimal power flow decisions)
	- renewable system control (learn optimal control policies under uncertainty)
- challenges of incorporating MDP
	- state space explosion = high computational cost (capacities, storage levels, fuel prices, policy states)
	- huge uncertainty + long planning horizon = hard to learn
	- curse of dimensionality
	- unknown transition probabilities (esp. for black swan events which are rare and fundamentally unpredictable)
- opportunities for incorporating MDP
	- stochastic capacity expansion where policy learned is contingent on uncertainty
	- learning-based investment / transition planning where policy learned is adaptive
	- real options valuation capturing value of waiting and flexibility as energy investment is irreversible
	- climate risk integration with weather uncertainty, extreme events, climate volatility
- hybrid approach
	- outer loop = MDP decides investment
	- inner loop = OR solves dispatch (dynamic programming)

black swan events = dynamic propagation problem, not just static optimization
- trigger:
	- immediate operational disruption
	- medium-term investment response
	- long-term structural transformation
- this is sequential decision under uncertainty (= MDP)
- current methods:
	- scenario analysis (solve cases separately, but discrete)
	- stochastic programming (scenario tree subject to multiple future branches, but only known branches)
	- sensitivity analysis (change input and resolve, but not dynamic)
- need for endogenous adaptive decision-making
	- state = energy system configuration
	- action = investment decision
	- transition = shock process + system evolution
	- reward = system cost
	- what is optimal policy given shocks may occur?
- questions to answer
	- how does adaptive investment policy differ from static planning under rare shocks?
		- state = installed capacity vector + shock state variable
		- action = investment decision
		- transition = shock arrival modelled as rare Poisson process
		- reward = dispatch cost solved using existing framework (e.g. PyPSA)
	- how can we quantify resilience value of a system?
		- define shocks (e.g. gas price spike, technology ban, policy change)
		- RL learns investment policy
		- c.f. deterministic optimization
	- what is the value of keeping flexible capacity?
- hybrid robust optimization + MDP
	- robust optmization models worst case shocks
	- MDP models adaptation (resilience analysis)

Resources:
[Investment Under Uncertainty](https://www.jstor.org/stable/j.ctt7sncv)
[Reinforcement Learning and Stochastic Optimization: A unified framework for sequential decisions](https://castle.princeton.edu/wp-content/uploads/2019/10/Powell-Reinforcement-Learning-and-Stochastic-Optimization.pdf)
