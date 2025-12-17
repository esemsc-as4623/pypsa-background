#pypsa #gb #marine #renewables 

https://www.sciencedirect.com/science/article/pii/S0306261923007778

- waves and tidal can be temporally and spatially offset from other variable renewables (e.g. solar, wind)
	- ocean energy profiles are more consistent
	- tidal range not considered
- framework to quantify potential benefit of including higher proportions of ocean energy within energy system
- economic dispatch model
	- model hourly supply-demand matching for range of sensitivity runs
	- adjust proportion of ocean energy within generation mix
- scenario:
	- 2030 case study of Great Britain
	- installed wave or tidal stream capacities from 100 MW to 10 GW
- findings: ocean energy
	- increases renewable dispatch
	- reduces dispatch costs
	- reduces generation required from fossil fuels
	- reduces system carbon emissions
	- reduces price volatility
	- captures higher market prices

gaps in literature filled
- dispatch modelling of hourly power system operation for scenarios including wave and tidal deployments
- temporally detailed profiles for spatially diverse sites for wave and tidal generation
- framework to investigate power system benefits associated with hourly offsetting of variable renewable generation profiles
- quantify impact of wave and tidal deployments

model inputs
- historical timeseries of demand and variable renewable availability
- scaling factors for future energy scenario demand
- installed capacities for current and future generation mix
- marginal costs for each form of generation
- net transfer capacities to represent transmission constraints

model validation
- historical year of data inputs
- 2019 GB scenario

model base cases
- future scenario data inputs
- 2030 GB future energy scenario

sensitivity analysis
- future scenario data inputs
- increasing proportion of ocean energy within future generation mix (replace w/ offshore wind only / onshore + offshore wind + solar)
- keeping total MWh renewable energy constant

model settings
- 9 zones (per National Grid transmission boundaries)
- linear optimal power flow equations
- perfectly competitive wholesale market
- perfect foresight of load and generation availability
- generation bids based on short-term (fuel and carbon) costs
- market clears at price of marginal generator at every hourly time step

validation
- developed based on 2019 GB power system
- c.f. 2019 and historical generation and market data to validate model outputs and understand how effectively it represents GB system
- metrics:
	- proportion of generation from renewables VS fossil fuels
	- marginal price compared to historical day-ahead market price for same year

ocean energy sensitivity
- run baseline 2030
- total amount of renewables available is kept constant b/w scenarios
- only proportion of generation from different technologies is changed
- different technologies have different capacity factors; this allows for fair exploration of system benefits of wave and tidal due purely to offsetting of resource within existing variable generation
- two cases
	- offshore wind case = reduce the capacity of offshore wind in proportion to wave and/or tidal added
	- wind and solar case = reduce both onshore + offshore wind + solar in proportion to wave and/or tidal added (relative capacities of wind and solar are adjusted based on total energy availability from each technology)

metrics
- PyPSA outputs
	- dispatch volumes $g$ (split per model zone $n$, generation type $r$, hourly snapshot $t$)
	- marginal prices $\lambda$ (split per zone $n$ and hourly snapshot $t$)
- metrics
	- proportion of annual generation from certain generator or from a technology category (sum of generator(s) over total dispatch volume over full year)
	- total carbon emissions from generation dispatched over the one year horizon (sumproduct of fossil fuel generation dispatched x emissions factors)
	- renewable curtailments (generation input to model - dispatch volumes output from model)
	- total cost of dispatch over full year (sumproduct of generation dispatched at each hourly snapshot and system marginal price for each hourly snapshot)
	- average marginal price of electricity (weighted average of all hourly marginal prices)
	- price volatility (std. dev. of all hourly marginal prices)
	- price capture for specific form of generation (energy-weighted average of all hourly marginal prices specific to availability of technology category)

data
- load
- generation
- marginal costs
- variable renewable profiles (availability)
- storage and flexibility
- transmission

results
- with more ocean energy (0-2GW)
	- small reduction in gas dispatch and increase in renewable dispatch
	- volume of curtailed renewable generation decreases
		- offsetting of wave and tidal with wind and solar, resulting in more consistently available renewable profile
	- overall reduction in annual carbon emissions and dispatch costs
		- more expensive and polluting generation is displaced
	- reduction in annual dispatch costs and carbon emissions continues as proportion of ocean generation increases
		- reduction in carbon emissions = almost linear pattern
		- fossil fuel dispatch (= carbon emissions) linearly decrease but only impact total cost of dispatch when enough fossil fuel dispatch is replaced to result in reduction in marginal cost for one or more time periods
	- average marginal price of electricity reduces slightly
		-  max and min marginal prices do not change
		- number of hours at min and max consistently reduce, reducing price volatility
- offshore wind is better correlated with demand than onshore wind or solar PV
	- substituting more offshore wind will have less benefit
- note: economic dispatch modelling does not fully represent all of the complexities of wholesale electricity trading
- price cannibalization = marginal price clears at 0 because demand can be met from renewables and storage of renewables for some hours
	- ongoing trend in high renewable energy markets
	- trend is forecast to increase in higher renewable scenarios
- wave and tidal capture higher prices than wind and solar, resulting in higher revenues per MWh
	- higher price capture occurs due to offsetting in resource profiles b/w different technologies
	- if wave/tidal is available at times of low wind/solar, they will capture higher marginal price
	- also due to lower installed capacity of wave/tidal c.f. wind/solar

economic dispatch modelling is a simplification
- optimizes dispatch costs for every hour of one year
- perfect foresight of demand and generation availability
- each type of generation bids into perfectly competitive market based on short-run costs
- does not capture financial or technical constraints on generators
	- need to recover long-term costs
	- must-run or ramping constraints for large thermal units
	- inertial constraints to ensure proper frequency response for system
- volatility of gas and electricity prices in recent years
- uncertain about hydrogen-to-electricity generation and grid-scale batteries

Repo: [[EVOLVE]]
