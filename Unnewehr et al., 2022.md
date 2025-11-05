#resolution #europe #pypsa #renewables #benchmark 

https://ieeexplore.ieee.org/document/9769123

references [[Frysztacki et al., 2021]]

validation study
- what effect does spatial clustering of transmission grids have on generation mix in optimization models?
- how well can EUR energy mix be backcast?
- how well can cross-border power flows be backcast?

network
- ENTSO-E grid map
- HVAC, HVCD, buses

demand
- country-level demand on ENTSO-E transparency website
- processed by Open Power System Data platform
- spatial resolution of demand to match network topology
- heuristic allocation using GDP and population
	- household demand in area proportional to population
	- industrial demand in area proportional to GDP
	- weight of GDP and population based on linear regression of per-country data

supply
- conventional
	- power plant matching
	- all assigned to nearest bus
- renewables
	- Markstammdatenregister (MaStR) for Germany
	- Open Power System Data for neighbors
	- IRENA capacity report for remaining
	- assigned to nearest bus and clustered per bus
- time series
	- hydro: ERA5 reanalysis
	- solar: ECMWF data + installed capacities
	- wind: ECMWF with offshore and onshore

clustering
- all lines mapped to same voltage (380 kV)
- nodes with one neighboring node (i.e. dead end) removed
- HVDC links in parallel or series merged to one link
- k-means clustering to spatial resolution
	- reduce nodes by geography, weighted by demand and conventional capacity
	- low network resolution can underestimate congestion due to renewable generation

study
- case 1: high resolution (1037 nodes) w/ within-country and cross-border network bottlenecks
- case 2: high resolution w/ fixed cross-border flow constrained to historical lines per ENTSO-E (sum of cross-border lines fixed to historical flow)
- case 3: low resolution (37 nodes, 1 per country) w/o within-country bottlenecks; only transmission capacities b/w countries are limited
- case 4: low resolution w/ constrained import / export volumes b/w countries per historical ENTSO-E (sum of energy flowing over all cross-border lines fixed to historical volume)

results
- modelled solar and onshore wind can be reproduced well in all 4 cases
- difference in offshore wind by resolution
- small capacity (biomass, waste) not reproduced well
- gas used for peak load coverage apparent in case 2 (high resolution, cross-border flow constraint)
- nuclear and hydro (usually base loads) reproduced acceptably but hourly generation varies from ENTSO-E significantly