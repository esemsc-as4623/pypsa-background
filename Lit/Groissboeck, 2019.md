#pypsa #osemosys 

https://www.sciencedirect.com/science/article/pii/S1364032118307743?pes=vor&utm_source=clarivate&getft_integrator=clarivate#bib49

unfulfilled functions in a single model
- distinguish between hot and cold startups or number of starts per day
- gaseous, liquid, and thermal energy distribution
	- required detailed physical attributes (e.g. temperature, pressure, velocity, and mass flow)
	- currently consider linear and predefined constant loss factors
- advanced or alternative objective functions (e.g. maximizing profit or efficiency)
	- creating Pareto curves
- short-term budget constraints
- rolling foresight
- limit the use of one fuel at a time
- technology aging and degradation
- environmental conditions (e.g. temperature, relative humidity)
- storage model with state-of-charge, depth-of-discharge
- maintenance planning, planned outages
- time-varying CapEx, OpEx
	- considered in TEMOA and OSeMOSYS
- risk appetite of investors
- system reliability indicators

top performing models:
- small time-steps
	- PyPSA
		- optimal power flow
		- feasible ramp rates
		- minimum up- and down-time
		- startup costs
- long-term planning
	- Switch
	- TEMOA
	- OSeMOSYS
		- multiyear investment
		- year-varying CapEx, OpEx, budget, emission constraints

specific shortcomings
- PyPSA
	- unable to consider spinning reserve or forecasting errors within dispatch procedure
- OSeMOSYS & TEMOA
	- do not incorporate power flow or unit commitment