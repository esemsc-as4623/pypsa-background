### Temporal Structure and Chronology

| **Concept**                 | **OSeMOSYS**                                                                                                                                                                                                    | **PyPSA**                                                                            | **Translation Challenge**                                                                                                                                                    |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Temporal representation     | Representative timeslices via hierarchical SEASON → DAYTYPE → DAILYTIMEBRACKET structure; timeslices are not chronologically ordered, they represent the proportion of a year that exhibits a certain condition | Chronologically ordered snapshots; can represent full year or representative periods | **Critical for storage:** OSeMOSYS cannot track true sequential state-of-charge dynamics; storage level transitions occur between _representative_ periods, not actual hours |
| Storage state tracking      | Hierarchical: StorageLevelYearStart, StorageLevelSeasonStart, StorageLevelDayTypeStart, StorageLevelDayTypeFinish                                                                                               | Sequential state_of_charge at each snapshot                                          | OSeMOSYS aggregates storage behavior within representative periods; PyPSA tracks actual chronological trajectories                                                           |
| Variable renewable profiles | CapacityFactor[r,t,l,y] — single value per timeslice (representative)                                                                                                                                           | p_max_pu can vary at each snapshot (chronological)                                   | Offshore wind variability and correlation with demand/storage needs is smoothed in OSeMOSYS                                                                                  |
| Cyclic boundary conditions  | Implicit year-end return to starting storage level                                                                                                                                                              | Optional cyclic_state_of_charge                                                      | Different handling of inter-annual storage behavior                                                                                                                          |
| Intra-timeslice variability | Lost through aggregation                                                                                                                                                                                        | Preserved if snapshots have sufficient resolution                                    | OSeMOSYS timeslice structure cannot capture short-duration wind ramps or storage cycling within a representative period                                                      |
### Storage Architecture

| **Concept**                      | **OSeMOSYS**                                                                                                                                          | **PyPSA**                                                                                                           | **Translation Challenge**                                                                                                                                                                          |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Storage-technology coupling      | Storage is a separate entity linked via binary TechnologyToStorage and TechnologyFromStorage parameters to distinct charging/discharging technologies | StorageUnit integrates storage with charge/discharge as a single component; alternatively Store + Link combinations | Efficiency losses must be embedded in the linking technologies' InputActivityRatio/OutputActivityRatio in OSeMOSYS, whereas PyPSA has explicit efficiency_store and efficiency_dispatch parameters |
| Round-trip efficiency            | No explicit parameter; must be modeled through separate charging/discharging technology efficiencies                                                  | Explicit efficiency_store × efficiency_dispatch                                                                     | Harmonization requires careful accounting of where losses occur                                                                                                                                    |
| Self-discharge / standing losses | No explicit parameter                                                                                                                                 | standing_loss parameter available                                                                                   | Cannot directly represent in OSeMOSYS without workarounds                                                                                                                                          |
| Storage duration specification   | Implicit through ratio of storage capacity (energy) to StorageMaxChargeRate/StorageMaxDischargeRate (power)                                           | Can specify max_hours directly or separate power/energy capacities                                                  | Different parameterization philosophies                                                                                                                                                            |
### Capacity-Activity Relationships

| **Concept**                     | **OSeMOSYS**                                                                            | **PyPSA**                                                   | **Translation Challenge**                                                                |
| ------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| CapacityToActivityUnit[r,t]     | Explicit conversion factor between capacity (power) and activity (energy) units         | Implicit; power and energy have standard unit relationships | Must ensure unit consistency; OSeMOSYS allows arbitrary unit systems                     |
| Availability vs Capacity Factor | Separate AvailabilityFactor[r,t,y] (annual) and CapacityFactor[r,t,l,y] (per timeslice) | Combined in p_max_pu time series                            | OSeMOSYS distinguishes planned outages (availability) from instantaneous capacity limits |
### Investment Timing and Multi-Period Structure

| **Concept**                   | **OSeMOSYS**                                                                    | **PyPSA**                                                                                                                                              | **Translation Challenge**                                                               |
| ----------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| Multi-year investment         | Native multi-year optimization over YEAR set with endogenous capacity expansion | Originally single-period dispatch; capacity expansion requires extensions (e.g., pypsa.optimize.create_model with multi-invest or external frameworks) | Comparing like-for-like requires careful scoping of investment vs operational decisions |
| Investment timing within year | Investment at year boundary; capacity available for full year                   | Investment typically for a single representative operational period                                                                                    | Different treatment of when capacity becomes available                                  |
| Residual capacity             | Explicit ResidualCapacity[r,t,y] parameter for pre-existing assets              | Handled through p_nom with p_nom_extendable=False                                                                                                      | Conceptually similar but different parameterization                                     |
### Offshore-Specific Edge Cases

| **Scenario**                                        | **OSeMOSYS Behavior**                                                                                             | **PyPSA Behavior**                                                          | **Implication**                                   |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------- |
| High offshore wind penetration with battery storage | Timeslice aggregation smooths wind variability; may underestimate curtailment and storage cycling requirements    | Chronological dispatch captures true curtailment needs and cycling patterns | Models may recommend different storage capacities |
| Coincident low wind + high demand events            | May not be captured if extreme events are averaged into representative timeslices                                 | Can explicitly include if snapshots cover extreme periods                   | Reliability assessment differs                    |
| Diurnal storage cycling (e.g., 4-hour battery)      | NetChargeWithinDay tracks daily net charge across representative day types, but intra-day dynamics are aggregated | Full intra-day charge/discharge dynamics if hourly snapshots used           | Short-duration storage utilization may differ     |
| Seasonal storage (e.g., hydrogen)                   | Hierarchical Season structure can track seasonal patterns, but through representative periods                     | Requires extended snapshot coverage or representative period selection      | Different approaches to long-duration storage     |
| Offshore wind capacity credit for reserve margin    | ReserveMarginTagTechnology allows fractional contribution (e.g., 0.1 for wind)                                    | No built-in reserve margin mechanism                                        | Policy constraint not directly translatable       |
| HVDC grid connection losses                         | Would model as technology efficiency via InputActivityRatio/OutputActivityRatio                                   | Can model explicit HVDC Link with efficiency and electrical properties      | Network representation differs fundamentally      |
### Additional Structural Differences

|**Concept**|**OSeMOSYS**|**PyPSA**|**Translation Challenge**|
|---|---|---|---|
|Fuel/carrier abstraction|FUEL set is generic — can represent any energy vector or abstract commodity|Carriers have defined types with physical interpretations (AC, DC, H2, etc.)|OSeMOSYS more flexible but less physically explicit|
|Trade between regions|Trade[r,rr,l,f,y] with TradeRoute binary parameter|Link components with explicit capacity, efficiency, and optionally electrical properties|OSeMOSYS trade is energy-balance based; PyPSA can include network physics|
|Minimum generation constraints|No minimum stable generation parameter|p_min_pu available (though often used with unit commitment)|Cannot directly enforce must-run or minimum output in standard OSeMOSYS|
|Curtailment tracking|Implicit; difference between potential and actual production|Can be explicitly tracked through slack variables or comparing p_max_pu to dispatch|Different approaches to quantifying curtailment|
|Accumulated demand|AccumulatedAnnualDemand[r,f,y] for demands without time-of-use profile|No direct equivalent; would need constraint on total energy|Different demand specification options|
### 1. Electrical Network Physics

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Reactive power (q)|Explicit for Generator, Load, Line, Transformer, ShuntImpedance|Not modeled|**Not translatable**|HVDC connections from offshore wind farms involve reactive power compensation; cannot assess VAR requirements|
|Voltage magnitude/angle|v_mag_pu, v_ang at each Bus|Not modeled|**Not translatable**|Offshore substations require voltage regulation; stability analysis not possible|
|AC power flow equations|KVL/KCL with impedance-based flow|Energy balance only|**Not translatable**|Cannot model electrical losses as function of loading; transport model only|
|Line impedance|r, x, g, b parameters|Not modeled|**Not translatable**|Long submarine cables have significant impedance; losses approximated only|
|Transformer parameters|tap_ratio, tap_position, phase_shift|Not modeled|**Not translatable**|Offshore transformer platforms not explicitly represented|
|ShuntImpedance|Conductance g, susceptance b|Not modeled|**Not translatable**|Reactive compensation for cable capacitance not captured|
|Thermal limits from impedance|s_nom with voltage-dependent limits|Capacity limits only|**Partially translatable**|Can set transfer capacity but not voltage-dependent derating|
### 2. Unit Commitment and Operational Constraints

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Binary commitment status|committable, status variable|Not modeled (LP only)|**Not translatable**|Backup gas turbines' start-up behavior not captured|
|Minimum up/down time|min_up_time, min_down_time|Not modeled|**Not translatable**|Thermal plant cycling constraints ignored|
|Start-up/shut-down costs|start_up_cost, shut_down_cost|Not modeled|**Not translatable**|Underestimates cost of wind variability management|
|Ramp rate limits|ramp_limit_up, ramp_limit_down|Not modeled|**Not translatable**|Cannot assess whether backup generation can follow wind ramps|
|Minimum stable generation|p_min_pu (with UC)|No minimum output constraint|**Not translatable**|Must-run constraints for grid-forming converters not captured|

**Edge Case**: Offshore wind with gas backup — PyPSA can model start-up costs incurred when wind drops suddenly; OSeMOSYS treats gas plant as infinitely flexible, underestimating system costs and potentially oversizing wind or undersizing backup.
### 3. Temporal Structure and Chronology

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Temporal representation|Chronologically ordered snapshots|Representative timeslices (TIMESLICE aggregated from SEASON × DAYTYPE × DAILYTIMEBRACKET)|**Structural disagreement**|Wind variability correlation with demand/storage lost in aggregation|
|Snapshot weightings|Multi-category (objective, store, generator) with flexible values|Single YearSplit[l,y] uniformly applied|**Not translatable**|Cannot weight peak demand periods differently for costs vs energy balance|
|Intra-timeslice dynamics|Preserved at snapshot resolution|Lost through aggregation|**Structural disagreement**|Short-duration wind ramps and storage cycling within representative period not captured|
|Sequence of events|Explicit chronological order|No inherent ordering within timeslice categories|**Structural disagreement**|Multi-day low wind events ("Dunkelflaute") may not be captured if averaged|
|Extreme event representation|Can include specific hours/days|Smoothed into representative periods|**Structural disagreement**|Coincident low wind + high demand events potentially underweighted|

**Edge Case**: 4-hour battery with offshore wind — PyPSA with hourly snapshots captures intra-day arbitrage value accurately; OSeMOSYS with daily timebrackets may aggregate away the price spread that justifies short-duration storage, leading to different storage sizing recommendations.
### 4. Storage Architecture and Dynamics

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Storage entity structure|StorageUnit (integrated power-energy) or Store + Link (decoupled)|STORAGE set linked to technologies via TechnologyToStorage, TechnologyFromStorage binary parameters|**Structural disagreement**|Different mental models; OSeMOSYS requires explicit charge/discharge technologies|
|Round-trip efficiency|Explicit efficiency_store × efficiency_dispatch|Implicit via charging/discharging technology InputActivityRatio/OutputActivityRatio|**Translatable with care**|Must correctly distribute losses across linking technologies|
|Self-discharge / standing losses|standing_loss parameter|Not modeled|**Not translatable**|Long-duration storage (hydrogen, compressed air) losses ignored|
|Energy-power coupling|StorageUnit: e_nom = p_nom × max_hours; Store: independent|Implicit via capacity and charge/discharge rates|**Translatable with care**|Different parameterization requires careful mapping|
|State-of-charge tracking|Sequential state_of_charge at each snapshot|Hierarchical: StorageLevelYearStart, StorageLevelSeasonStart, StorageLevelDayTypeStart/Finish|**Structural disagreement**|OSeMOSYS tracks representative period transitions, not actual chronological SOC|
|Cyclic boundary conditions|Optional e_cyclic per Store/StorageUnit|Implicit return to StorageLevelStart[r,s] at year end|**Partially translatable**|Inter-annual storage behavior handled differently|
|Minimum state of charge|e_min_pu|MinStorageCharge[r,s,y]|**Translatable**|Similar functionality|
|Storage capital cost|Per-energy capital_cost for Store; per-power for StorageUnit|CapitalCostStorage[r,s,y] per energy unit|**Translatable with care**|Must clarify whether cost is per-MWh or per-MW|

**Edge Case**: Seasonal hydrogen storage for offshore wind — OSeMOSYS SEASON structure can track inter-seasonal storage levels, but through representative days, not actual 8760 hours. If hydrogen is charged during windy summer days and discharged during calm winter nights, OSeMOSYS captures the seasonal pattern but loses intra-seasonal dynamics. PyPSA with full-year hourly resolution captures complete cycling behavior but at computational cost.

**Edge Case**: Battery degradation — Neither model natively captures cycle-dependent degradation. OSeMOSYS OperationalLifeStorage is fixed; PyPSA has no built-in degradation. For offshore battery systems with high cycling, both models may overestimate lifetime value.
### 5. Cost Accounting and Economic Conventions

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Objective function basis|Annualized costs (default) or NPV with explicit investment periods|NPV of total system cost over model period|**Harmonizable via post-processing**|Different cost metrics reported; same physical solution possible|
|Salvage value treatment|Not included by default; requires manual handling|Explicit SalvageValue[r,t,y] credited at end of operational life|**Harmonizable via post-processing**|Long-lived offshore assets (40-year foundations) significantly affected by salvage treatment|
|Depreciation method|Not specified|DepreciationMethod[r]: sinking fund (1) or straight-line (2)|**OSeMOSYS-specific**|Affects salvage value calculation for offshore infrastructure|
|Intra-period discounting|Applied to snapshot-weighted costs|Operational costs discounted to beginning of year|**Harmonizable via parameter adjustment**|Minor impact for most applications|
|Fixed operating costs|capital_cost only; fixed O&M typically added to marginal cost|Explicit FixedCost[r,t,y] per unit capacity|**Translatable**|Offshore O&M costs (vessel access, maintenance windows) can be represented in both|
|Capital recovery|Implicit in annualized cost|Spread across operational life via discounting|**Harmonizable via post-processing**|Different reporting of capital expenditure timing|

**Edge Case**: Short planning horizon (2025-2035) with 30-year offshore wind turbines — Salvage value treatment dramatically affects economics. OSeMOSYS credits residual value; PyPSA without adjustment does not. Models may agree on capacity but report very different total costs.
### 6. Policy and Regulatory Constraints

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Reserve margin|Not built-in; requires custom GlobalConstraint|Explicit ReserveMargin[r,y], ReserveMarginTagTechnology[r,t,y], ReserveMarginTagFuel[r,f,y]|**OSeMOSYS → PyPSA: requires manual implementation**|Offshore wind capacity credit (e.g., 10-20%) explicitly handled in OSeMOSYS|
|RE production target|Via GlobalConstraint on carrier production|Explicit REMinProductionTarget[r,y], RETagTechnology[r,t,y], RETagFuel[r,f,y]|**Translatable in both directions**|RE targets can drive offshore wind deployment in both models|
|Emission limits|Via GlobalConstraint with co2_emissions carrier attribute|AnnualEmissionLimit[r,e,y], ModelPeriodEmissionLimit[r,e]|**Translatable**|Carbon constraints captured in both|
|Emissions penalty|Via GlobalConstraint shadow price or explicit carrier cost|Explicit EmissionsPenalty[r,e,y]|**Translatable**|Carbon price propagation to dispatch differs in implementation|
|Exogenous emissions|Not built-in|AnnualExogenousEmission[r,e,y], ModelPeriodExogenousEmission[r,e]|**OSeMOSYS-specific**|Accounting for emissions outside model boundary|

**Edge Case**: Capacity adequacy with offshore wind — OSeMOSYS reserve margin constraints can directly limit offshore wind's contribution to firm capacity via ReserveMarginTagTechnology < 1. PyPSA requires custom constraint formulation, risking different interpretations of capacity credit methodology.
### 7. Capacity and Activity Modeling

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Capacity-to-activity conversion|Implicit; power (MW) × time (h) = energy (MWh)|Explicit CapacityToActivityUnit[r,t] allowing arbitrary unit systems|**Translatable with care**|Must ensure unit consistency; OSeMOSYS more flexible but error-prone|
|Availability vs capacity factor|Combined in p_max_pu time series|Separate AvailabilityFactor[r,t,y] (annual planned outages) and CapacityFactor[r,t,l,y] (instantaneous availability)|**Partially translatable**|Offshore wind scheduled maintenance (availability) vs wind resource (capacity factor) can be distinguished in OSeMOSYS|
|Lumpy investment|Via p_nom_extendable with custom constraints|CapacityOfOneTechnologyUnit[r,t,y] triggers MILP|**Translatable**|Offshore wind farms typically built in discrete project sizes|
|Model period activity limits|Via GlobalConstraint|TotalTechnologyModelPeriodActivityUpperLimit[r,t], TotalTechnologyModelPeriodActivityLowerLimit[r,t]|**OSeMOSYS-specific**|Cumulative energy extraction limits (e.g., gas reserves)|
|Residual capacity|p_nom with p_nom_extendable=False|Explicit ResidualCapacity[r,t,y]|**Translatable**|Pre-existing offshore capacity handled similarly|

**Edge Case**: Offshore wind with distinct failure modes — OSeMOSYS can separate planned maintenance (AvailabilityFactor = 0.95) from wind-dependent availability (CapacityFactor profile). PyPSA combines both into p_max_pu. If maintenance windows correlate with low-wind periods (summer maintenance during calm weather), OSeMOSYS's separation may be more accurate.

### 8. Demand Modeling

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Demand specification|Load component with p_set time series|SpecifiedAnnualDemand[r,f,y] × SpecifiedDemandProfile[r,f,l,y]|**Translatable**|Onshore demand profile shapes storage requirements|
|Accumulated demand|Not built-in|AccumulatedAnnualDemand[r,f,y] for demands without time profile|**OSeMOSYS-specific**|Useful for annual energy targets without hourly profile|
|Elastic demand|Via custom constraints or sector coupling|Not built-in|**Neither model native**|Demand response to offshore wind availability requires extensions|

### 9. Multi-Regional and Trade

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Regional representation|Bus components (can aggregate)|REGION set|**Translatable with aggregation**|Multiple buses can map to one region|
|Inter-regional transfer|Link with efficiency, capacity, and optionally electrical properties|Trade[r,rr,l,f,y] with TradeRoute enabling parameter|**Partially translatable**|PyPSA can model HVDC physics; OSeMOSYS uses simple efficiency loss|
|Multi-port conversion|Link with bus0, bus1, bus2... and efficiency, efficiency2...|Via MODE_OF_OPERATION with different input/output ratios|**Translatable with care**|Offshore hydrogen production with multiple outputs handled differently|

**Edge Case**: Meshed offshore grid — PyPSA can model interconnected North Sea grid with AC/DC power flow; OSeMOSYS represents only energy transfer limits between regions. Network congestion patterns and optimal power flow routing cannot be captured in OSeMOSYS.
### 10. Technology and Fuel Abstraction

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Mode of operation|Implicit in component type|Explicit MODE_OF_OPERATION set with InputActivityRatio[r,t,f,m,y], OutputActivityRatio[r,t,f,m,y]|**OSeMOSYS-specific**|Flexible fuel switching (e.g., electrolyzer producing H2 or providing grid services) explicitly modeled|
|Carrier definition|Carrier component with co2_emissions, max_growth, max_capacity|FUEL set with emissions via EmissionActivityRatio|**Partially translatable**|OSeMOSYS emissions tied to technology activity, not fuel consumption directly|
|Technology-fuel linkage|Via component's carrier attribute|Via InputActivityRatio, OutputActivityRatio matrices|**Translatable**|Different parameterization philosophies|

### 11. Investment Timing and Pathway Optimization

|**Feature**|**PyPSA**|**OSeMOSYS**|**Translation Status**|**Offshore Relevance**|
|---|---|---|---|---|
|Multi-year optimization|Via explicit investment periods with build_year, lifetime tracking|Native via YEAR set with OperationalLife[r,t]|**Translatable**|Both can model 2025-2050 pathways|
|Investment period weighting|Explicit weightings per investment period|Implicit via DiscountRate[r]|**Partially translatable**|Different control over inter-temporal trade-offs|
|Automatic retirement|Based on lifetime parameter|Based on OperationalLife and ResidualCapacity trajectory|**Translatable**|Long-lived offshore foundations handled similarly|
|Myopic vs perfect foresight|Configurable|Default perfect foresight|**Configurable in both**|Investment timing under uncertainty handled similarly|
### 12. Summary: Critical Translation Challenges for Offshore Wind + Storage

| **Challenge**                   | **Nature**                 | **Impact on Results**                                                                   | **Mitigation**                                                                                              |
| ------------------------------- | -------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Chronological fidelity          | Structural disagreement    | Storage sizing and dispatch patterns may differ; curtailment underestimated in OSeMOSYS | Use high temporal resolution in PyPSA; carefully design OSeMOSYS timeslice structure to capture key periods |
| Storage efficiency architecture | Implementation difference  | Minor if carefully mapped                                                               | Document where losses are applied in each model                                                             |
| Reserve margin methodology      | OSeMOSYS-specific          | Wind capacity credit handled differently                                                | Implement equivalent constraint in PyPSA                                                                    |
| Salvage value treatment         | Cost accounting difference | Large impact for long-lived assets in short horizons                                    | Post-processing adjustment or explicit salvage in PyPSA                                                     |
| Unit commitment                 | PyPSA-specific             | OSeMOSYS underestimates flexibility costs                                               | Acceptable for long-term planning; use PyPSA for operational validation                                     |
| Network physics                 | PyPSA-specific             | OSeMOSYS cannot assess transmission constraints, losses                                 | Use OSeMOSYS for generation adequacy; PyPSA for network studies                                             |
| Self-discharge losses           | PyPSA-specific             | Long-duration storage economics affected                                                | Approximate via reduced efficiency in OSeMOSYS                                                              |
