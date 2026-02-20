#optimization #renewables #storage 

https://www.pfenninger.org/publications-pdf/Pfenninger%20et%20al_2014_Energy%20systems%20modeling%20for%20twenty-first%20century%20energy%20challenges.pdf

note: this paper predates PyPSA and OSeMOSYS (which was very new)

four categories
1. energy systems optimization models = models covering entire energy system, primarily using optimization methods, with primary aim of providing scenarios of how system could evolve
2. energy systems simulation models = models covering entire energy system, primarily using simulation techniques, with primary purpose of providing forecasts of how system may evolve
3. power systems and electricity market models = models focused on electricity system, ranging from optimization/scenarios to simulation/prediction
4. qualitative and mixed-methods scenarios = scenarios relying more on qualitative or mixed methods rather than detailed mathematical models

four challenges
1. resolving time and space
2. balancing uncertainty and transparency
3. addressing growing complexity
4. integrating human behaviour and social risks and opportunities

scenario planning = develop scenarios about possible future evolution
- pioneered by RAND corporation as "future-now thinking"

ESMs allow development of scenarios
- made possible the formalization of scattered knowledge about complex interactions
- structured way of thinking about implications of changes
- allow policy-makers to explicitly state views on steering the system to fulfil policy goals

energy problems + opportunities:
- security
- affordability
- resilience
- pollution
- sustainability
- new technologies to market
- build competitive new industries
- fuel rapid economic growth with new sustainable energy production

challenges:
- rise of flexible demand driven by new technologies (e.g. smart meters, distributed generation)
- importance of electrification and intermittent supply, requiring more temporal detail
- paradigm of distributed energy and varying renewable resource potential, requiring more spatial detail

traditional ESMs unable to address competing claims (e.g. actual cost of renewables due to intermittency)
- scenarios can produce aggregate cost figures and decarbonization targets
- scenarios cannot answer Qs about feasible configurations of real renewables-based energy system or possible bottlenecks
- recent modelling efforts to deliver spatial + temporal resolution

ways to divide models:
- simulation / forecast VS optimization / scenarios
	- simulation = predictive
	- optimization = normative
- planning VS operation
	- planning = capacity expansion
	- operation = dispatch
	- increasing need for both in one model
- snapshots VS pathways
	- snapshot = of desired end state
	- pathway = decisions needed to reach end state

type 1: energy system optimization models
- large, bottom-up
- based on detailed description of technical components of energy system
- make simplifications to remain tractabe
	- e.g. limit to nationally aggregated build, yearly or seasonally averaged demand-supply balancing
- e.g. MARKAL/TIMES family of models, MESSAGE, OSeMOSYS
- represent possible evolutions of energy system model on national / regional / global basis over several decades w/o saying anything about likelihood of evolutions
- linear optimization to minimize total energy system cost
- recently: non-linear and MILP
- hybrid models
	- link technologically rich bottom-up models with top-down general equilibrium economic models (with marginal abatement cost curves or production functions)
	- characterize economy-wide movements in response to energy system changes
- challenge: resolving details in time and space
	- balance model resolution w/ data availability + computational tractability
	- coarse = sensible + deal with temporal fluctuations that are not important
		- fossil fuel or nuclear heavy systems can assume baseload or dispatchable at will so output has negligible dependence on fluctuating external influences
	- renewable potential and cost differs in space; intermittency and availability differs in time
		- variation in time can be reduced if fluctuations can be balanced out by spatial distribution
		- amount of storage needed = important driver of system cost

type 2: energy system simulation models
- simulation = generate possible futures, focus on predicting system's likely evolution
- contrast with rigid mathematical formulation of type 1
- build modularly, incorporate a range of methods (some submodules can be optimization-based)
- e.g. NEMS, PRIMES, LEAP
	- submodules iteratively solved by central integrating module
	- individual submodules flexibly implemented
		- e.g. demand based on macroeconomic model
	- high complexity; results difficult to understand
	- submodules represent independent agents
	- model finds equilibrium solution for supply, demand, cross-border trade, emissions
- challenge: uncertainty + transparency
	- epistemic uncertainty = if more / better data will reduce uncertainty
	- aleatory uncertainty = if uncertainty cannot be reduced further
		- differentiate b/w deterministic and stochastic methods
		- deterministic (e.g. Monte Carlo) uncertainty analysis to examine effect of changes in model inputs on model outputs
		- stochastic = specify distributions for params and let model incorporate into decisions (e.g. stochastic MARKAL / MESSAGE = original LP + non-linear risk functions)
		- vary only small subset of params
		- unexpected sources of uncertainty
	- fit for purpose?
		- ESM = not physically verifiable against observable phenomena
		- seen as possible storyline rather than fundamental truth
		- situations modelled cannot be fully observed / measured and so cannot be properly validated [[DeCarolis et al., 2012]]
		- intransparent = inner workings not described in detail
		- inaccessible = analyses not reproducible
		- high decision stakes + high system uncertainty
	- ESMs examine implications of assumptions made by modellers
		- technology cost, performance, economic development, policies

type 3: power systems and electricity market models
- normative-optimization, predictive-simulation modeling
- more detail in temporal variation for constant balance b/w suppy and demand
- e.g. WASP, PLEXOS
- dynamic programming for generation expansion planning
- challenge: complexity and optimization across scales
	- simplification + low resolution
	- over-optimized = risk of diminishing returns as overhead of maintaining system grows + increased vulnerability to unexpected shocks
	- spatial + temporal scale can vary drastically

type 4: qualitative + mixed-methods
- back-of-the-envelope quantitative reasoning + qualitative judgement
- challenge: capturing human dimension

resolving time and space
- load duration curves or capacity factors
- time slices with representative days and seasons
- real time series of production potential
- more time slices can gloss over correlation b/w real weather and miss system-defining extreme points

uncertainty, accessibility, reproducibility
- complex energy models are no better than simple ones in predictive power
- small model = rigorous uncertainty and sensitivity analysis
- e.g. TEMOA implements MGA
- make source code and data open
- make transparency a design goal
- utilize free software tools
- develop test systems for verification exercises
- interoperability b/w models

complexity and optimization across scales
- consider different time scales with different levels of detail
- planning step + operational step
- operational time scale = decisions on how to operate available system to satisfy given demand
- simplified heuristics at planning time scale
- challenge: reveal big picture but reach wrong conclusion due to simplification
- multi-scale / multi-level methods in other domains
- exploit properties of the problem to simplify its solution in fully integrated model
	- e.g. spatial economics: adaptive approach to decide which geographical zones to cluster and which to keep disaggregated to minimize model error
- complex systems can exhibit emergent effects from interaction b/w constituent parts --> sudden and dramatic changes
	- agent-based models capture interactions

integrating social theory