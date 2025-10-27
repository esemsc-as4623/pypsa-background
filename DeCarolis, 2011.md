#optimization #mga

https://www.sciencedirect.com/science/article/pii/S0140988310000721?via%3Dihub

modelling to generate alternatives (MGA)
- systematically explore feasible, near-optimal solution space to develop alternatives that are maximally different in decision space but perform well in modelled objectives
- challenge preconceptions / assumptions

energy-economy optimization objectives
- prediction of future quantities
- prescriptive analysis for planning
- generation of insight related to policy design and implementation

issue: future uncertainties
- structural uncertainty = imperfect and incomplete nature of equations describing system
- parametric uncertainty = assumed value of model inputs (data, values, stochasticity)
	- scenario analysis is insightful if represents range of inputs, assigned a subjective probability, mutually exclusive and exhaustive
	- monte carlo simulation yields distribution of outputs that reflects uncertainty in input (represented as probability distributions)
		- quantitative measure of risk / dispersion in model results
		- each model realization within monte carlo assumes (a priori) a particular state of the world based on input values sampled; parameter uncertainty propagated through the model
		- joint distribution updated after each decision to reflect partial resolution of uncertainty and effect of learning

algorithm (hop-skip-jump)
- obtain initial optimal solution by any method
- add user-specified amount of slack to value of objective function(s)
- encode adjusted objective function value(s) as additional upper bound constraint(s)
- formulate new objective function that minimizes weighted sum of decision variables that appeared in previous solutions
- iterate the re-formulated optimization
- terminate MGA when no significant changes to decision variables are observed in the solutions