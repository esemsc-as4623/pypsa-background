#pypsa #europe 

1. *Least-cost capacity expansion with $CO_2$ constraints and emission pricing

`snakemake -call results/test-elec/networks/base_s_6_elec_.nc --configfile config/test/config.electricity.yaml`

output: base_s_6_elec_.nc

| output                                  | how it is stored                                                                                                                                                                                                                        | insight it provides                                                                                                  |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| network structure and topology          | - n.buses (6 clustered regions)<br>- n.lines (transmission corridors b/ regions w/ optimal capacities)<br>- n.links (HVDC connections w/ expansion decisions)<br>- n.generators (optimal mix of generation technologies per region)     | - transmission expansion<br>- regional specialization                                                                |
| capacity investment decisions (MW)      | - n.generators.p_nom_opt (how much of each technology to build)<br>- n.lines.s_nom_opt (transmission capacity investments)<br>- n.storage_units.p_nom_opt (batter/hydrogen storage sizing)                                              | - technology portfolio<br>- investment priorities<br>- grid bottlenecks                                              |
| operational dispatch schedules (hourly) | - n.generators_t.p (generation dispatch by technology and region)<br>- n.lines_t.p0 (power flows on transmission lines)<br>- n.storage_units_t.p (storage (dis-)charging patterns)<br>- n.loads_t.p (electricity demand profiles)       | - renewable curtailment<br>- storage utilization<br>- system flexibility                                             |
| economic analysis                       | - n.objective (total annual system cost)<br>- n.buses_t.marginal_price (electricity prices by region and hour)<br>- n.generators.marginal_cost (operating cost by technology)<br>- n.lines.capital_cost (transmission investment costs) | - cost-effectiveness<br>- price volatility<br>- regional price differences                                           |
| environmental performance               | - annual emissions = (n.generators_t.p * n.generators.carrier.map(emission_factors)).sum()<br>- renewable_share = renewables_generation / total_generation<br>- capacity_factors = n.generators_t.p.mean() / n.generators.p_nom_opt     | - system $CO_2$ footprint<br>- renewable penetration<br>resource utilization                                         |
| impact of dynamic line rating           | - n.lines_t.s_max_py (time varying transmission capacities)<br>- utilization_rates = n.lines_t.p0.abs() / (n.lines.s_nom_opt * n.lines_t.s_max_pu)                                                                                      | - weather-based capacity management<br>- congestion patterns<br>- economic benefit of dynamic VS static line ratings |

| File                                   | Design Choices | Config Choices                                                                                                            |
| -------------------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `build_line_rating.py`                 |                |                                                                                                                           |
| `add_transmission_projects_and_dlr.py` |                |                                                                                                                           |
| `simplify_network.py`                  |                |                                                                                                                           |
| `build_electricity_demand_base.py`     |                |                                                                                                                           |
| `process_cost_data.py`                 |                |                                                                                                                           |
| `cluster_network.py`                   |                | N = number of buses a network should be reduced to<br>number of country subnetworks $\leq$ N $\leq$ total number of nodes |
| `determine_availability_matrix.py`     |                |                                                                                                                           |
| `build_powerplants.py`                 |                |                                                                                                                           |
| `build_renewable_profiles.py`          |                |                                                                                                                           |
| `add_electricity.py`                   |                |                                                                                                                           |
| `prepare_network.py`                   |                |                                                                                                                           |
| `solve_network.py`                     |                |                                                                                                                           |

