https://www.transitionzero.org/technology/open-source-interoperability-integrating-osemosys-and-pypsa-for-power-system-analysis

#pypsa #osemosys 

OSeMOSYS = good for long-term, integrated energy planning
- optimize capacity expansion and technology choice across multiple sectors and decades
- what technology to build and when to build them to meet demand cost-effectively considering resource availability and policy constraints
- simplified representation of electricity grid
	- abstract away hourly resolution and network constraints for reliable system operation

PyPSA = high-resolution power-system analysis
- detailed electricity networks
- hourly dispatch
- generator unit commitment
- complex interplay of renewable variability and storage across regions
- implications of grid bottlenecks
- precise hourly costs and emissions
- short-term horizon
- requires pre-defined capacity expansion pathways

TransitionZero hopes to build a unified, interoperable model

https://forum.openmod.org/t/your-opinion-on-current-open-models/692/4
- mathematical programming models = short, comprehensible, readily extensible to other optimization paradigms (e.g. stochastic programming)
	- e.g. OSeMOSYS
- object-oriented models = self-contained classes with design benefits, sophisticated technology / entity characterizations, rule-based dispatch, genetic algorithms
	- e.g. PyPSA