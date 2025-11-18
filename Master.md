# Tags
#pypsa #marine #europe #mathematics #osemosys #deeplearning #optimization #renewables #storage #mga #gnn #benchmark 
# Literature
[x] = good resource

| Reference                          | Title                                                                                                                                                                                                                | Year |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- |
| [[Lavidas et al., 2025]]           | Marine renewables in Energy Systems: Impacts of climate data, generators, energy policies, opportunities, and untapped potential for 100% decarbonised systems                                                       | 2025 |
| [[Groissboeck, 2019]]              | Are open source energy system optimization tools mature enough for serious use?                                                                                                                                      | 2019 |
| [[Hoffmann et al., 2024]]<br>[x]   | A review of mixed-integer linear formulations for framework-based energy system models                                                                                                                               | 2024 |
| [[Moksnes and Usher, 2023]]        | Quantifying the relative importance of the spatial and temporal resolution in energy systems optimisation model                                                                                                      | 2023 |
| [[Mao et al., 2024]]               | A review of the energy storage system as a part of power system: Modelling, simulation and prospect                                                                                                                  | 2024 |
| [[Frysztacki et al., 2021]]<br>[x] | The strong effect of network resolution on electricity system models with high shares of wind and solar                                                                                                              | 2021 |
| [[Ringkjøb et al., 2018]]          | A review of modelling tools for energy and electricity systems with large shares of variable renewables                                                                                                              | 2018 |
| [[Unnewehr et al., 2022]]<br>[x]   | The value of network resolution -- A validation study of the European energy system model PyPSA-Eur                                                                                                                  | 2022 |
| [[DeCarolis, 2011]]<br>[x]         | Using modelling to generate alternatives (MGA) to expand our thinking on energy futures                                                                                                                              | 2011 |
| [[Varbella et al., 2024]]          | PowerGraph: a power grid benchmark dataset for graph neural networks                                                                                                                                                 | 2024 |
| [[Pfenninger et al., 2014]]        | Energy systems modeling for twenty-first century energy challenges<br><br>[link](https://www.sciencedirect.com/science/article/pii/S1364032114000872)                                                                | 2014 |
| [[Giarola et al., 2022]]           | MUSE: An open-source agent-based integrated assessment modelling framework<br><br>[link](https://www.sciencedirect.com/science/article/pii/S2211467X22001584)                                                        | 2022 |
| [[McCollum et al., 2020]]          | Energy modellers should explore extremes more systematically in scenarios<br><br>[link](https://www.nature.com/articles/s41560-020-0555-3)                                                                           | 2020 |
| [[Chang et al., 2021]]             | Trends in tools and approaches for modelling the energy transition<br><br>[link](https://www.sciencedirect.com/science/article/pii/S0306261921002476)                                                                | 2021 |
| [[Gissey et al., 2019]]            | Value of energy storage aggregation to the electricity system<br><br>[link](https://www.sciencedirect.com/science/article/pii/S0301421519300655?via%3Dihub)                                                          | 2019 |
| [[Brown et al., 2018]]             | Synergies of sector coupling and transmission reinforcement in cost-optimised, highly renewable European energy system<br><br>[link](https://www.sciencedirect.com/science/article/pii/S036054421831288X?via%3Dihub) | 2018 |
# Definitions
- ***model predictive control*** = control algorithm that utilizes model predictions to optimize performance through model prediction, rolling optimization, and feedback correction, allowing for effective management of uncertainties in dynamic systems
- methodologies
	- ***optimization*** = determine the system that is optimal according to a metric
	- ***equilibrium*** = determine the system as a result of interacting balanced components (cannot be validated)
	- ***simulation*** = determine the system given some rules (often bottom-up)
- uncertainty
	- ***deterministic optimization*** (perfect foresight) = all future states (exogenous parameters) are known at the beginning of the model horizon
	- ***stochastic optimization*** = all future states along an uncertainty tree are known, including probabilities of each branch
	- ***myopic optimization*** (rolling horizon) = decisions in period *y* are taken under some assumptions about the future; move to period *y+1* and repeat, with possibly altered assumptions about subsequent periods (*y+2, ...*)
- event categories
	- transient events
	- disruptive drivers
	- unexpected outcomes
- ***levelized cost of electricity*** = net present value of costs over lifetime divided by total electricity demand; indicator for the cost of one unit of electricity generated by a power plant
- ***load duration curve*** = describes the number of hours in the year that load was greater than or equal to a given level
- ***merit order dispatch rule*** = in order of increasing variable cost, assigns technologies to load blocks of decreasing duration until either all load blocks are satisfied or all generating capacity is exhausted
- ***unit commitment*** = optimization problem where production of a set of generators is coordinated to meet demand at minimum cost; storing energy at scale is difficult and start-up / shut-down timeline requires notice but variation in consumption needs to be met

# Conferences
| Conference                      | Date               | Location        | Notes                                                                                                                                                          |
| ------------------------------- | ------------------ | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ABM4Energy 2026                 | 30-31 March 2026   | Vienna, Austria | Agent-Based Modelling for Energy Economics and Energy Policy<br>https://events.hifis.net/event/2964/overview                                                   |
| Geilo Winter School x NAIC 2026 | 18-23 January 2026 | Geilo, Norway   | Uncertainty Quantification for Scientific Computing Intelligent Systems<br>https://www.sintef.no/projectweb/geilowinterschool/2026-uncertainty-quantification/ |
| EGU2026                         | 3-8 May 2026       | Vienna, Austria | European Geosciences Union<br>https://www.egu26.eu/                                                                                                            |

# Companies
- Resources for the Future -- think tank in Washington DC
- International Institute for Applied Systems Analysis (IIASA)
- Agora Energiewende
- d-fine (Management Consulting)
- Fraunhofer IEG (Research Institution for Energy Infrastructures and Geotechnologies)
- Fraunhofer IEE (Institute for Energy Economics and Energy System Technology)

# People
- Daniel Huppmann, TU Wien
- Behnam Zakeri, IIASA
- Matthew Gidden (Open Software)
- Natsuo Kishimoto (Open Software)
- Anthony Papavasiliou (OR)
- Tom Brown, TU Berlin (PyPSA)
- Fabian Neumann, TU Berlin (PyPSA)
- Joseph DeCarolis (U.S. Energy Information Administration)
- Maximilian Parzen (Open Energy Transition)

# Useful Links
- Open energy platform: https://openenergyplatform.org
- Open energy modelling initiative: https://openmod-initiative.org
- UK Department for Business, Energy, & Industrial Strategy: https://my2050.energysecurity.gov.uk/
- Mackay Carbon Calculator: https://mackaycarboncalculator.energysecurity.gov.uk/
- Electricity Demand: https://github.com/open-energy-transition/Awesome-Electricity-Demand?tab=readme-ov-file#europe
- Map Your Grid: https://mapyourgrid.org
- Energy Transition Model: https://www.energytransitionmodel.com
- Our World in Data (Renewable Energy): https://ourworldindata.org/renewable-energy
- U.S. Department of Energy Energy Storage Handbook: https://www.sandia.gov/ess/publications/doe-oe-resources/eshb
- Open energy benchmark: https://openenergybenchmark.org/
- Energy System Catapult: https://es.catapult.org.uk/