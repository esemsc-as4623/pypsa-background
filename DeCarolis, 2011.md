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

avoids cognitive bias associated with bottom-up construction
- reformulated objective function seeks to minimize appearance of decision variables from previous solutions
- energy futures are distinctly different from each other
- degree of difference in decision space depends on
	- variance in objective function coefficients
	- amount of prescribed slack
	- degree to which decision variables are constrained
- valuable to know if a modified set of objective function coefficients in the original non-MGA formulation can uniquely reproduce the results from a MGA solution

interpretations
- accounts for structural uncertainties e.g. unmodelled objectives
- represents a perturbation that can uniquely produce the MGA solution

other formulations
- partial equilibrium: maximize the deployment of a particular set of technologies within the cost-effective region surrounding the optimal
- decision support: explore decision space by altering model objectives

limitations
- choice of slack parameter is subjective and has strong effect on alternative solutions
	- relate slack to key scenario indicators, or
	- perform parametric sensitivity analysis of slack parameter
- not possible to unique obtain particular MGA solution
- does not assign subjective probabilities to alternative solutions; cognitive heuristics still play a role
	- do not implicitly assign equal probability to alternatives
	- evaluate based on desirable features that are exogenous to energy economy optimization models
	- develop quantitative measures of degree of difference

recommendation
- sensitivity analysis to estimate partial rank correlation coefficients and identify input parameters with greatest effect on key model outputs
- joint distribution of key uncertain parameters for multistage stochastic optimization framework
- apply MGA to stochastic optimization solution to test robustness of optimal hedging strategy