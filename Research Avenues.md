
- probabilistic / stochastic risk modelling
- Monte Carlo simulations

# Self-critical Questions
- model uncertainty
	- is the approach appropriate?
	- are results dependent on the methodology?
- parameter uncertainty
	- how much confidence can you have on input assumptions?
- model horizon, temporal / spatial disaggregation
	- what is the intended scope of analysis?
	- end-of-horizon effect?
- model simplifications for numerical tractability and comprehensibility
	- what are the appropriate trade-offs between high level of detail vs big picture?
	- assumption or result?
- system boundaries, model closure
	- are the assumptions valid?

# Validation
- sensitivity analysis
	- structured variation of key input parameters to understand the impact on results
	- easy but never comprehensive
- multi-criteria analysis
	- include multiple dimensions in the objective function, solve model with different weights
	- still prone to modelling artifacts
- modelling to generate alternatives
	- re-solve a model to get a different solution within some additional bounds
	- elegant but requires substantial effort to implement
	- resource: [[DeCarolis, 2011]]

# Criticisms of Energy Transition Models
- energy transitions are to complex to model
	- technologies
	- inter-dependencies
	- actors
	- institutions
- future is full of uncertainties; modelling creates a false impression that we know what may happen
- oversimplification
- models have failed in the past to simulate trends
- models cannot always represent disruptions, technological revolutions, etc
- societal and individual behavior and lifestyle changes are not easy to quantify
- assumptions may be biased by the view of individual or institution
- unclear cause-effect relationships
- black box models are not validated; different models produce different insights; models lack practical view (too theoretical)
- perfect foresight = assume that everything is known and optimize deterministically; BUT, we have
	- demand uncertainty
	- fuel price volatility
	- technology learning uncertainty
	- policy changes
	- climate variability
	- extreme weather events

1. Markov Decision Process
	- sequential, adaptive, uncertain, path-dependent
	- state = installed capacity, demand, fuel price, carbon price, technology cost, storage levels, grid constraints, climate conditions
	- actions = investment and operational decisions (build, retire, invest, expand, implement demand response, etc.)
	- transition model = represent demand evolution, fuel price dynamics, technology learning curves, policy shifts, weather patterns
	- reward = social welfare - system cost - emissions damage
	- discount factor = represent time preference, intergenerational equity, social discount rate
	- output = contingent strategies (e.g. if gas prices rise, build renewables faster; if demand stagnates, delay nuclear; if storage cost drops, accelerate electrification)