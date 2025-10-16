#mathematics

## General
Source: [[Hoffmann et al., 2024]]
### Model Overview
The model is a linear minimization problem focused on optimizing the capacity and operation costs across various components and scenarios.

### Equations
#### Objective Function

$$

\text{min} \left( \sum_{c} \sum_{i} \sum_{r} \left( C^{\text{capex}}_{c,r,i} + \sum_{s} \sum_{t} p_{s} c^{\text{op}}_{c,i,r,s,t} x^{\text{op}}_{c,i,r,s,t} \right)  \right)

$$

- **Purpose**: Minimizes net present value of capacity and operation costs for all components, investment periods, regions, load scenarios, and time steps.

#### CAPEX Calculation

$$

C^{\text{capex}}_{c,r,i} = \sum^{i}_{j=i-lt_{c}/|i|+1} c^{\text{cap}}_{c,r,j} \cdot x^{\text{commis}}_{c,r,j} \frac{\text{PVIFA}(d,|i|)}{\text{PVIFA}(d,lt_{c})} \cdot \frac{1}{(1+d)^{j \cdot |i|}}

$$

- **Description**: Computes capacity-related net present value, transforming commissioning costs into annuity costs using PVIFA.

#### Flow Conservation

$$

\sum_{g \in \mathbb{G}, i \in \mathbb{I}, r \in \mathbb{R}, s \in \mathbb{S}, t \in \mathbb{T}} : f_{c,g,i,r,s,t} = 0

$$

- **Objective**: Ensures conservation of flow for commodities entering and leaving a hub.

#### Source and Sink Operations
- **Sources**:
  
$$

  f_{c,g,i,r,s,t} = x^{\text{op}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{source}} \cap \mathbb{M}^{g}
  
$$

- **Sinks**:
  
$$

  f_{c,g,i,r,s,t} = -x^{\text{op}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{sink}} \cap \mathbb{M}^{g}
  
$$

- **Description**: Links operations to commodity flows produced or consumed.

#### Conversion Units

$$

f_{c,g,i,r,s,t} = \gamma_{c,g,r,s,t} \cdot x^{\text{op}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{conv}} \cap \mathbb{M}^{g}

$$

- **Purpose**: Ties production/consumption to conversion units with conversion factor $\gamma$.

#### Storage and Transmission
- **Storage**:
  
$$

  f_{c,g,i,r,s,t} = x^{\text{op,dis}}_{c,i,r,s,t} - x^{\text{op,ch}}_{c,i,r,s,t} \quad \forall c \in \mathbb{M}^{\text{store}} \cap \mathbb{M}^{g}
  
$$

- **Transmission**:
  
$$

  f_{c,g,i,r,s,t} = x^{\text{op}}_{c,i,(r',r),s,t} - x^{\text{op}}_{c,i,(r,r'),s,t} \quad \forall c \in \mathbb{M}^{\text{trans}} \cap \mathbb{M}^{g}
  
$$

- **Details**: Manages flows in storage charge/discharge and transmission across regions.

### Constraints

$$

x^{\text{op}}_{c,i,r,s,t} \ge 0 \quad \forall c \in \mathbb{M}^{\text{source,sink,conv,store}}, i, r, s, t

$$

- **Function**: Non-negativity ensures operation limits are adhered to, considering capacity and time steps $\Delta t$.

#### State of Charge

$$

x^{\text{SOC}}_{c,i,r,s,t+1} = x^{\text{SOC}}_{c,i,r,s,t} + \eta^{\text{ch}}_{c,i,r,s,t} x^{\text{op,ch}}_{c,i,r,s,t} - \frac{x^{\text{op,dis}}_{c,i,r,s,t}}{\eta^{\text{dis}}_{c,i,r,s,t}}

$$

- **Role**: Links state of charge between time steps, maintaining non-negativity and capacity limits.

#### Transmission Capacity

$$

-x^{\text{cap}}_{c,i,r,s,t} \Delta t \le x^{\text{op}}_{c,i,(r',r),s,t} - x^{\text{op}}_{c,i,(r,r'),s,t} \le x^{\text{cap}}_{c,i,r,s,t} \Delta t

$$

- **Purpose**: Ensures net flow along transmission lines adheres to capacity limits, improving stability.

### Under the Hood
- **Objective**: Minimize operational and capacity costs effectively.
- **Equations**: Ensure flow conservation, respect operation, and transmission limits.
- **Constraints**: Non-negativity and capacity adherence for all components ensure realistic operational limits and enhance model stability.


## Best Principles

- variability
	- daily variations in supply and demand balanced by
		- short-term storage (e.g. batteries, pumped hydro, small thermal)
		- demand-side management
		- east-west grids over multiple time zones
	- weekly variations in supply and demand balanced by
		- medium-term storage (e.g. hydrogen, methane, thermal, hydro reservoirs)
		- continent-wide grids
	- seasonal variations in supply and demand balanced by
		- long-term storage (e.g. hydrogen, methane, thermal, hydro reservoirs)
		- north-south grids over multiple latitudes
- boundary conditions (avoid too many assumptions, let math decide the rest)
	- meed demand for energy services
	- reduce $CO_2$ emissions
	- conservative predictions for cost developments
	- no/minimal/optimal grid expansion
- linear optimization of annual system costs
	- minimize annual system costs = annualized capital costs + marginal costs
	- subject to
		- meeting energy demand at each node $n$ (e.g. region) and time $t$ (e.g. hour of year)
		- wind, solar, hydo (variable renewables) availability time series $\forall n, t$
		- transmission constraints b/w nodes, linearized power flow
		- installed capacity $\le$ geographical potentials for renewables
		- $CO_2$ constraint (e.g. 95% reduction compared to 1990 levels)
	- mostly greenfield investment optimization, multi-period with linear power flow
	- optimize generation + storage + transmission jointly b/c they are interacting

## PyPSA
### Constraints
1. Nodal energy balance
		Demand $d_{i,t}$ at each node $i$ and time $t$ is always met by generation / storage units $g_{i,s,t}$ at the node or from transmission flows $f_{l,t}$ on lines attached at the node (Kirchhoff's Current Law): $$ \sum_{s} g_{i,s,t} - d_{i,t} = \sum_{l} K_{il}f_{l,t} $$
2. Generation availability
		Generator / storage dispatch $g_{i,s,t}$ cannot exceed availability $G_{i,s,t} * G_{i,s}$ made up of per unit availability $0 \le G_{i,s,t} \le 1$ multiplied by the capacity $G_{i,s}$
		Capacity is bounded by the installable potential $\hat{G}_{i,s}$ $$ 0 \le g_{i,s,t} \le G_{i,s,t} * G_{i,s} \le G_{i,s} \le \hat{G}_{i,s}$$
3. Storage consistency
4. Kirchhoff's Laws for Physical Flow
5. Transmission line thermal limits
6. Global constraints on $CO_2$ and transmission volumes