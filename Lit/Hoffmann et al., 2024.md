#pypsa #europe #mathematics #optimization 

https://www.sciencedirect.com/science/article/pii/S2666792424000283?via%3Dihub

optimization-based frameworks for ESM
- TIMES
- ETHOS.FINE
- PyPSA

review of 63 optimization-based frameworks for ESM
focus on mathematical implementation
meta-review of 68 existing literature reviews

missing thus far
- how to set up modelling frameworks
- how to model specific features

open source linear and mixed-integer linear optimization frameworks
- comparable
- basic mathematical model as largest common denominator

techno-economic perspectives
- top-down
	- focus on economic laws and treating technical aspects in less detailed manner
- bottom-up
	- considers differentiated energy systems and network topologies and pay less attention to market mechanisms

modelling techniques
- optimization
- simulation

model dimensions
- commodities = medium that "flows" between components
	- energy forms
	- energy carriers
	- raw materials
- spatial resolutions = regions, locations, nodes
	- congestion free commodity flows between demand and supply
	- multi-nodal systems explicitly consider capacity-related transport restrictions of commodities between nodes
	- transmission losses 
- temporal resolutions = consider demand and supply situations at discrete time steps
	- operational variables defined for each time step must stay within capacity limits
	- crucial for storage which is dependent on transient energy supply and demand cycles over time
	- uncertainty and transformation processes (i.e. transient system designs over time)
		- operational reliability is a priority (e.g. extreme scenarios)
		- long-term planning where status quo system is gradually replaced
- stochastic scenarios = uncertainty
	- discrete scenarios with predefined probabilities of occurrence
	- uncertainties in model parameters can be considered simultaneously
	- model many system critical events (e.g. unexpected outages)
	- adds additional stage to optimization models
		- deterministic equivalent programs
	- each scenario has its own chance of occurrence and no preceding or succeeding scenario
	- used for robustness but not energy storage
- transformation pathways
	- brownfield analysis = starting with an existing system
	- capture stepwise transformation of the system using multiple investment periods during which capacities of energy technologies gradually change
	- transient capacity stock (decommissioning at end-of-life)
	- ETHOS.FINE, OSeMOSYS, REMix, TIMES
- components = sinks, sources, converters, storage, transmission lines
	- capacity variable
		- capacities are variable in capacity expansion models
			- finding cost-minimal energy system designs
		- capacities are fixed in dispatch or unit commitment models
			- minimize operational costs of existing system
	- operational variables (several, one for each time step)
		- operational variables must not surpass installed capacity of the respective component
	- source = feed commodities into system
	- sinks = withdraw commodities from system
	- converters = convert two or more commodities into one another, linked via conversion rates
	- storage = consumes commodities at one point in time to release them later
		- state of charge = sum of charges and withdrawals over time
	- transmission units = connect regions, sink in one and simultaneously source in another
	- most components contribute to overall cost of energy system
		- maximum input/output capped by respective capacities
	- some components (e.g. demand sinks) do not have variable net capacity but fixed demand must be satisfied at any point in time
	- hubs within a spatial region maintain flow conservation (i.e. regarded as congestion-free commodity flow)

basic model formulation
- linear minimization problem
| Symbol | Description |
|---------|-------------|
| $\mathbb{M}^{\mathit{source}}$ | subset of components representing sources |
| $\mathbb{M}^{\mathit{sink}}$    | subset of components representing sinks |
| $\mathbb{M}^{\mathit{conv}}$   | subset of components representing conversion units |
| $\mathbb{M}^{\mathit{store}}$   | subset of components representing storage units |
| $\mathbb{M}^{\mathit{trans}}$   | subset of components representing transmission units |
| $\mathbb{M}^{g}$        | components associated with a commodity (a good) in $g$ |
| $\mathbb{G}$          | commodities |
| $\mathbb{I}$            | investment periods |
| $\mathbb{R}$           | regions |
| $\mathbb{S}$           | scenarios |
| $\mathbb{T}$           | time steps |
| $\mathbb{P}$           | (typical) periods (e.g., typical days) |
| $\mathit{C}^{\mathit{CAPEX}}$ | capital expenditures of commissioned capacities |
| $\mathit{f}$            | commodity flow variable |
| $\mathit{x}^{\mathit{op}}$     | operation-rate variable |
| $\mathit{x}^{\mathit{op,bin}}$ | binary variable indicating whether a component is active (1) or not (0) |
| $\mathit{x}^{\mathit{op,bin,sd}}$ | binary variable indicating whether a component is deactivated (1) or not (0) |
| $\mathit{x}^{\mathit{op,bin,su}}$ | binary variable indicating whether a component is activated (1) or not (0) |
| $\mathit{x}^{\mathit{op,ch}}$  | operation-rate variable representing charging |
| $\mathit{x}^{\mathit{op,dis}}$ | operation-rate variable representing discharging |
| $\mathit{x}^{\mathit{op,net}}$ | net operation-rate (mass-flow) variable |
| $\mathit{x}^{\mathit{SOC}}$    | variable representing the state of charge of a storage |
| $\mathit{x}^{\mathit{cap}}$    | installed capacity variable |
| $\mathit{x}^{\mathit{commis}}$ | commissioned capacity |
| $\mathit{c}^{\mathit{cap}}$        | annualized net capacity cost |
| $\mathit{c}^{\mathit{op}}$         | operation cost |
| $\mathit{d}$           | discount factor |
| $\mathit{lt}$           | lifetime |
| $\mathit{MDT}$    | minimal down-time |
| $\mathit{MUT}$    | minimal up-time |
| $\mathit{r}^{\mathit{down}}$      | maximum down-ramping rate |
| $\mathit{r}^{\mathit{up}}$         | maximum up-ramping rate |
| $\Delta t$         | time-step length |
| $\gamma$            | conversion factor for conversion of one commodity into another |
| $\lambda$            | dimensionless weighting factor with values between 0 and 1 |
| $\eta^{\mathit{ch}}$          | charging efficiency of a storage |
| $\eta^{\mathit{dis}}$         | discharging efficiency of a storage |
| $\eta^{\mathit{sd}}$          | self-discharge rate |
| $\theta$             | net capacity factor |
| $\mathit{CAPEX}$  | capital expenditures |
| $\mathit{LODF}$     | line outage distribution factor |
| $\mathit{PTDF}$     | power transfer distribution factor |
| $\mathit{PVIFA}$   | present value interest factor of annuity |


- equations
$$

\text{min} \left( \sum_{c} \sum_{i} \sum_{r} \left( C^{\text{capex}}_{c,r,i} + \sum_{s} \sum_{t} p_{s} c^{\text{op}}_{c,i,r,s,t} x^{\text{op}}_{c,i,r,s,t} \right)  \right)

\
\

\text{s.t.} \quad \forall c \in \mathbb{M}, i \in \mathbb{I}, r \in \mathbb{R} : C^{\text{capex}}_{c,r,i}

\
\

= \sum^{i}_{j=i-lt_{c}/|i|+1} c^{\text{cap}}_{c,r,j} \cdot 
x^{\text{commis}}_{c,r,j} \frac{\text{PVIFA}(d,|i|)}{\text{PVIFA}(d,lt_{c})} \cdot \frac{1}{(1+d)^{j \cdot |i|}}

$$
objective function, minimizing the net present value of the capacity and operation costs of all components, all investment periods, all spatial regions, and for all load scenarios, and every time step.

$C^{\text{capex}}_{c,r,i}$ = capacity-related net present value of capacity expenditures (CAPEX) of the capacity stock of a component throughout an investment period and its capacity in that investment period. Uses the present value interest factor of annuity (PVIFA) to transform commissioning costs to annuity costs and subsequently to costs per investment period.

$x^{cap}_{c,r,i} = \sum^{i}_{j=i-lt_{c}/|i|+1} \cdot x^{commis}_{c,r,j}$ = component's capacity in the investment period
$$

\sum_{g \in \mathbb{G}, i \in \mathbb{I}, r \in \mathbb{R}, s \in \mathbb{S}, t \in \mathbb{T}} : f_{c,g,i,r,s,t} = 0

$$
maintains the flow conservation for all commodities entering and leaving a hub
- defined for each commodity, time step, region, investment period, scenario
$$

f_{c,g,i,r,s,t} = x^{\text{op}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{source}} \cap \mathbb{M}^{g}

$$
links the operation of sources to the respective flows of commodities produced.
$$

f_{c,g,i,r,s,t} = -x^{\text{op}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{sink}} \cap \mathbb{M}^{g}

$$
links the operation of sinks to the respective flows of commodities consumed.
$$

f_{c,g,i,r,s,t} = \gamma_{c,g,r,s,t} \cdot x^{\text{op}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{conv}} \cap \mathbb{M}^{g}

$$
links the operation of conversion units to the production or consumption of commodities.
- as conversion units link multiple commodities, their operation is linked to each involved commodity with an individual conversion factor $\gamma$
$$

f_{c,g,i,r,s,t} = x^{\text{op,dis}}_{c,i,r,s,t} - x^{\text{op,ch}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{store}} \cap \mathbb{M}^{g}

$$
refers to the flow of storage components in charging direction, defined for each time step, region, investment period, scenario
$$

f_{c,g,i,r,s,t} = x^{\text{op}}_{c,i,(r',r),s,t} - x^{\text{op}}_{c,i,(r,r'),s,t} \quad \forall c \in \mathbb{M}^{\text{trans}} \cap \mathbb{M}^{g}

$$
refers to the flow of transmission components from region $r$ to region $r'$, defined for each time step, region, investment period, scenario
$$

\text{s.t.} \quad \forall c \in \mathbb{M}^{\text{source,sink,conv,store}}, i \in \mathbb{I}, r \in \mathbb{R}, s \in \mathbb{S}, t \in \mathbb{T} : x^{\text{op}}_{c,i,r,s,t} \ge 0

$$
ensures non-negativity of the components' variable operation and that none of the operations surpasses the components' net capacity
- maximum power output can deviate from the nominal net capacity
- amount of produced, consumed, converted, stored or transmitted commodities depends on length of discrete time steps, $\Delta t$
$$

x^{SOC}_{c,i,r,s,t+1} = x^{SOC}_{c,i,r,s,t} + \upeta^{ch}_{c,i,r,s,t} x^{op,ch}_{c,i,r,s,t} - \frac{x^{op,dis}_{c,i,r,s,t}}{\upeta^{dis}_{c,i,r,s,t}} \quad \forall c \in \mathbb{M}^{store}

$$
links states of charge between subsequent time steps, constrained to be non-negative and not surpass storage net capacity, guaranteed by $x^{SOC}_{c,i,r,s,t} \ge 0 \quad \forall c \in \mathbb{M}^{store}$ and $x^{SOC}_{c,i,r,s,t} \le x^{cap}_{c,i,r} \quad \forall c \in \mathbb{M}^{store}$.
$$

-x^{\text{cap}}_{c,i,r,s,t} \Delta t \le x^{op}_{c,i,(r',r),s,t} - x^{op}_{c,i,(r,r'),s,t},\quad x^{cap}_{c,i,r,s,t} \Delta t \ge x^{op}_{c,i,(r',r),s,t} - x^{op}_{c,i,(r,r'),s,t}
\
\quad \forall c \in \mathbb{M}^{trans}, r'

$$
ensures the net flow of commodities along a transmission line never exceeds the link’s net capacity.
$$
-x^{\text{cap}}_{c,i,r,s,t} \Delta t \ge x^{op}_{c,i,(r',r),s,t} + x^{op}_{c,i,(r,r'),s,t} \quad \forall c \in \mathbb{M}^{trans}, r'
$$
limits the flow in each direction to be at most as big as the built transmission line net capacity multiplied by the time step duration
- reduces model's numeric stability and convergence behavior by limiting its indifference regarding gross flows in opposite directions

extensions
- CAPEX usually grow non-linearly with capacity due to technological learning or (dis-)economies of scale
	- economies of scale = output-specific costs decrease with total output
		- marginal costs reduce with output or size
	- technological learning = tendency of a technology to become more mature over time
		- capacity specific costs decrease with total installed capacity
- ramping = gradient by which a component's operation can change over time (measure of inertia)
- minimum part load = operation must be either zero or above minimum load
- minimum up- and down-times = require components to remain active/inactive for minimum amount of time before changing operational status
- price elasticity = change in demand with change in price
	- discretize demand curve using series of energy sinks
	- energy sinks yield revenues in decreasing order
	- sink generating highest revenue per energy unit is supplied first
	- order of price segments is automatically kept by ESM
	- energy supply is increased up to a point at which the additional system costs equal the revenues generated from the elastic energy sink
- curtailment (load shedding) = reduction of load peaks
	- due to pure energy savings, grid bottlenecks, shortage of backup capacity
	- assumed that reduced load does not increase load during other time steps
- load shifting = avoid load peaks with advanced or postponed load catch-up during another time
- conversion b/w energy carriers as non-linear functions of part load efficiencies
	- modelled by piecewise linear functions
- storage losses = charging losses, discharging losses, self-discharge losses
- physical constraints, e.g. temperature and mass flow dependence in heating networks
- boundary conditions
	- potential = geographical / technological upper bounds and constraints
	- regulations, e.g. subsidy mechanisms
	- volume-related restrictions = components restricted in design and/or operation without a direct impact on cash flows
- risk management
	- n-1 criteria
	- reserve margins
	- firm capacities to ensure security of supply
- multi-criteria objectives (e.g. low cost, low emissions)
- spatial aggregation = related to grid topolgy
	- reduces number of network nodes
	- underestimates total annualized system costs by omitting some transmission bottlenecks
- technological aggregation = related to supply and demand technologies
	- overestimates total annualized system costs by averaging out most profitable generation sites with high potential for capacity expansion
- temporal aggregation
- investment pathway coupling
	- uncoupled investment periods = parallelization of each investment period
	- myopic foresight method = series of brownfield analyses in succession
	- backcasting = solve target system, define boundary conditions in last investment period, recursively repeat until first investment period is optimized
	- rolling-horizon = one optimization for two investment periods; design choices fixed for the first period in each pair; subsequent period re-evaluated in next run
	- all-at-once = consider entire horizon at once (perfect foresight)
	- investment inefficiencies; expensive, sub-optimal solutions
- decompose time / space / technology
	- sort variables and constraints by chosen dimension
	- model sparsity = opportunity for decomposition
	- master problem defines cost coefficients for sub-problems
	- sub-problems consist of problems with modified objective functions
	- sub-problems solved in parallel
	- solutions can be infeasible, feasible and suboptimal, or optimal
	- master problem adjusted until optimality criterion is met
- optimization limitations
	- technical
		- real systems face uncertain future demand; optimization frameworks have perfect information
		- rule-based energy management systems still more widely used b/c of low data requirement and simplicity
	- financial
		- optimization neglects some stakeholders and trade options b/w them
		- agent based models
			- autonomous decision-making entities
			- bottom-up energy system optimization
			- sub-model to account for market principles
		- bi-level programming
			- integration multiple agents into models with single hard-coupled optimization
			- optimization problem of one or many agents as side constraints for another
			- nested optimization = MILP with Karush-Kuhn-Tucker conditions
			- e.g. model interaction of price-setters and price-takers (Stackelberg pricing games)
	- environmental
		- $CO_2$ emissions, recycling, LCA, material flows
		- energy systems interact with the environment on many levels, direct and indirect
		- large-scale capacity expansion can cause feedback effects on micro-climates (e.g. wakes)
	- social
		- fairness, social acceptance, behavioral adaptation, political uncertainty
		- affordability, disproportionate benefits
		- subsidization schemes