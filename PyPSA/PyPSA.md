#pypsa 

[[PyPSA-EUR]]
[[EVOLVE]]
### Comparisons
[[PyPSA vs OSeMOSYS]]

(https://forum.openmod.org/t/your-opinion-on-current-open-models/692/4)
- PyPSA cannot perform optimization over multiple time horizons (c.f. OSeMOSYS, IKARUS, TIMES)
	- also a limitation for Oemof and Calliope
	- in the works

(https://www.sciencedirect.com/science/article/pii/S0306261923018652)
- Calliope, OSeMOSYS, PyPSA, TIMES have the potential to be used for evaluating a transition pathway but have not been used in this way with hourly time resolution over significant time slices
- TIMES
	- thorough scenario exploration: multi-cell modelling, pathway analysis, full-scale representation, market equilibrium dynamics
- Calliope
	- typically employed for scenario analysis w/ specific focus on electricity system
	- used to assess impact of inter-year variability in wind and PV on results rather than transition pathway

### Energy Transition Pathways
two approaches for optimizing the energy transition pathway
1. run snapshot model multiple times using an algorithm that optimizes the transition path and validates the system's operability
2. extend a snapshot model to represent the entire transition pathway (but few are sufficiently fast and adaptable to do this)
