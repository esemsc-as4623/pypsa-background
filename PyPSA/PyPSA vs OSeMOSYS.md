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

### Translation Planning

**Direct Mappings**:
- Bus → Region
- Carrier → Commodity/Fuel
- Generator → Technology (generation)
- Load → SpecifiedAnnualDemand
- StorageUnit → Technology (storage) with coupled capacity

**Complex Mappings**:
- Store + Link → Requires separate storage and conversion technologies
- Link (multi-port) → May require multiple Technology definitions
- Line/Transformer → Transmission constraints between regions

**Parameter Translation Challenges**:
- **Time Resolution**: PyPSA uses flexible snapshots; OSeMOSYS uses Timeslice structure (Season, DayType, DailyTimeBracket)
- **Capacity Units**: PyPSA uses MW/MWh; OSeMOSYS uses activity units (PJ/year typical)
- **Cost Structure**: PyPSA separates capital_cost and marginal_cost; OSeMOSYS uses CapitalCost, FixedCost, VariableCost
- **Efficiency**: PyPSA uses per-unit [0,1]; OSeMOSYS uses InputActivityRatio/OutputActivityRatio
- **Storage**: PyPSA couples energy/power in StorageUnit; OSeMOSYS has independent StorageLevelStart, ResidualStorageCapacity
- **Investment Periods**: PyPSA uses flexible years; OSeMOSYS uses YEAR set with explicit periods
- **Availability**: PyPSA uses p_max_pu time series; OSeMOSYS uses CapacityFactor by timeslice
- **Ramp Rates**: PyPSA has explicit ramp_limit_*; OSeMOSYS uses RampRate and RampingReset parameters

**Missing PyPSA Features in OSeMOSYS**:
- Reactive power (q) and AC power flow
- Voltage magnitude and angle
- Shunt impedances
- Tap ratios and phase shifts
- Multi-port links (OSeMOSYS has binary Mode of Operation)

**Missing OSeMOSYS Features in PyPSA**:
- Reserve margin requirements (ReserveMargin, ReserveMarginTagTechnology)
- Emission activities separate from energy (EmissionActivityRatio)
- Operational life distinct from technical life
- Capacity credit (CapacityFactor vs availability)
- Storage levels by timeslice

| PyPSA Component                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | OSeMOSYS Component                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Implementation                                                                                                                                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`bus`](https://docs.pypsa.org/latest/user-guide/components/buses/)<br>- fundamental node of network<br>- enforces energy conservation<br>- represent grid connection point or non-electric energy carriers or non-energy carriers in different locations<br>                                                                                                                                                                                                                                                                                                                      | `REGION.csv`                                                                                                                                                                                                                                                                                                                                                                                                                                                           | one REGION = one bus<br><br>[x] scenario/components/topology.py<br>                                                                                                                                                                         |
| [`snapshot`](https://docs.pypsa.org/latest/user-guide/design/#snapshots)<br>- represent time steps<br>- used to represent time-varying nature of network<br>- all time-dependent series quantities indexed by snapshots<br>- snapshot weightings applied to each snapshot so that snapshots represent more than one hour or fractions of one hour (default: 1.0)<br>- 3 categories of weightings:<br>1. objective = weight in objective function<br>2. store = state of charge change for stores and storage units<br>3. generator = used in global constraints and energy balance | `YEAR.csv`- discrete years in model<br>`SEASON.csv`- unique seasons in each year<br>`DAYTYPE.csv`- unique day types in each season<br>`DAILYTIMEBRACKET.csv`- unique time periods in each day<br>`TIMESLICE.csv`- unique season x daytype x timebracket (repeats for each year)<br>`YearSplit.csv`-<br>`DaySplit.csv`-<br>`DaysInDayType.csv`- <br>`Conversionls.csv`- <br>`Conversionld.csv`- <br>`Conversionlh.csv`- <br><br>#todo still unclear on what DaySplit is | snapshot = unique (YEAR, TIMESLICE) tuple<br>snapshot_weighting = Conversionlh x Conversionld x Conversionls for each (YEAR, TIMESLICE)<br><br>#todo how to deal with the three types of weightings?<br><br>[x] scenario/components/time.py |
| [`investment_periods`](https://docs.pypsa.org/latest/user-guide/design/#investment-periods)<br>- monotonically increasing integers of years<br>- by default, single investment period (overnight scenario)<br>- investment weightings applied to each investment period (default: 1.0)<br>- 2 columns:<br>1. objective = multiplied with all cost coefficients in objective function of investment period<br>2. years = elapsed time until subsequent investment period                                                                                                            | N/A                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                             |
| [`load`](https://docs.pypsa.org/latest/user-guide/components/loads/)<br>- attached to single bus<br>- represents demand<br>- (if $sign = 1$, models exogenous supply, e.g. a building with solar panels with supply > demand)<br>- $p>0$: load is consuming active power<br>- $q > 0$: load is consuming reactive power                                                                                                                                                                                                                                                            | `AccumulatedAnnualDemand.csv`<br>`SpecifiedAnnualDemand.csv`<br>`SpecifiedDemandProfile.csv`                                                                                                                                                                                                                                                                                                                                                                           | [x] scenario/components/demand.py<br><br>#todo how does PyPSA-EUR deal with $q$?                                                                                                                                                            |
| [`carrier`](https://docs.pypsa.org/latest/user-guide/components/carriers/)<br>- energy carrier of bus<br>- AC / DC / hydrogen / heat                                                                                                                                                                                                                                                                                                                                                                                                                                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | add all carriers to network<br><br>#todo                                                                                                                                                                                                    |
| [`generator`](https://docs.pypsa.org/latest/user-guide/components/generators/)<br>- attach to a single bus<br>- feed in power<br>- convert energy from their carrier to the carrier of the bus to which they attach<br>- (if $sign = -1$, models withdrawal, e.g. a battery charging)                                                                                                                                                                                                                                                                                              | `CapacityToActivityUnit.csv`<br>`CapacityFactor.csv`<br>`AvailabilityFactor.csv`<br>`OperationalLife.csv`<br>`ResidualCapacity.csv`<br>`InputActivityRatio.csv`<br>`OutputActivityRatio.csv`                                                                                                                                                                                                                                                                           | [. ] scenario/components/supply.py                                                                                                                                                                                                          |
|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                             |
