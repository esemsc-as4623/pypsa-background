#resolution #renewables #storage #europe #pypsa 

https://www.sciencedirect.com/science/article/pii/S0306261921002439

computational resource VS model accuracy
- need to capture weather-driven variability
- need to assess investment in generation / transmission and flexibility options over weather and demand situations across locations
- need at least hourly resolution
- consequences of clustering representative solutions

low resolution = suboptimal investment for wind and solar
- ignore power grid bottlenecks within regions underestimates system / network-related costs
- aggregating locations w/ different capacity factors overestimates costs

methodology to disentangle competing spatial effects (resource and network resolution)
- co-optimization of transmission, generation, storage at high spatial and temporal resolution
- linearized grid physics + existing transmission lines + realistic restrictions on grid reinforcement

data
- historical hourly load data for EUR from Open Power System Data project
- distributed according to population and GDP
- generation time series from historical wind and insolation from ERA5 reanalysis and SARAH2
- potentials based on land cover maps
- reproduced historical congestion (curtailment due to transmissions bottlenecks in Germany) in 2013-2018

network
- grid
	- k-means clustering of model nodes
		- geographical location weighted by average load and conventional capacity at substations
		- long transmission lines b/w regions left to be optimized
		- high density of nodes aggregated together
		- short lines are inexpensive to upgrade and rarely present bottlenecks
		- maintain topology of grid, coherent, independent of pre-defined dispatch patterns
	- desired number of nodes partitions b/w countries and synchronous zones
	- reconnect with major transmission corridors
	- demand, conventional generation, storage also aggregated to nearest network node
- renewables
	- case 1: simultaneous clustering (generation sites = transmission nodes)
	- case 2: clustering on site resolution (fix transmission nodes, increase generation and storage sites)
		- all wind and solar generators connected to central node
		- no network bottlenecks
	- case 3: clustering on network nodes (high resolution of generation sites, increase transmission nodes)

model
- optimize investments and operation for wind / solar + open cycle gas turbines, batteries, hydrogen storage, transmission, with flexibility of hydroelectric power plants
- perfect foresight at 3-hour resolution over historical load and weather, assuming 95% reduction in $CO_2$ emissions c.f. 1990
- vary the amount of new transmission that can be built to study grid reinforcement on results

results
- case 1: change from offshore to onshore wind with increasing resolution
	- apparent stability of total system costs is deceptive
	- sinking costs from high resource resolution countered by rising costs from network bottlenecks
	- different effects on technology mix
- case 2: costs decrease with increasing resolution
	- also related to change from offshore to onshore wind with increasing resolution
	- onshore stronger near coastlines
- case 3: increasing transmission nodes = higher costs as bottlenecks are visible
	- driven by generation and storage
	- transmission bottlenecks limit transfer of power from best sites to the load, forcing model to build onshore wind and solar more locally at sites with lower capacity factors
	- curtailment is low
	- battery and hydrogen storage increases as storage is used to balance variations to avoid overloading grid bottlenecks