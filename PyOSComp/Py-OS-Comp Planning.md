#pypsa #osemosys #benchmark 

# PyPSA Features Not Translated to OSeMOSYS

1. Reactive Power (q) Modeling
	- Reactive power variables `q` for all components (Generator, Load, Line, Transformer)
	- Reactive power constraints and balance equations
	- Voltage magnitude `v_mag_pu` and angle `v_ang` variables
	- Power factor constraints
	- Reactive power compensation (ShuntImpedance components)
	**Translation Approach**: All reactive power parameters and constraints are **ignored**. Only active power `p` is translated to OSeMOSYS activity/energy variables.
2. AC/DC Network Flow Constraints
	- Kirchhoff's Voltage Law (KVL) and Current Law (KCL) constraints
	- Line impedance (r, x) and admittance modeling
	- AC power flow equations (P = V²/Z, reactive power flow)
	- DC power flow linearization constraints
	- Voltage angle differences across lines
	- Line thermal limits based on impedance and voltage
	**Translation Approach**:
	- **Single-region**: All PyPSA buses aggregate to one OSeMOSYS REGION. Intra-regional network (Lines, Transformers) is abstracted away entirely.
	- **Multi-region**: User specifies bus→region mapping. Lines connecting different regions map to TradeRoute with capacity limits, but impedance and voltage constraints are lost.
3. Unit Commitment Features
	- Binary commitment status variables `status` (on/off)
	- Minimum up time constraints `min_up_time`
	- Minimum down time constraints `min_down_time`
	- Start-up costs `start_up_cost`
	- Shut-down costs `shut_down_cost`
	- Ramp rate limits `ramp_limit_up`, `ramp_limit_down`
	- Committable flag for generators `committable`
	**Translation Approach**: Unit commitment features are **ignored**. PyPSA generators with `committable=True` translate as standard technologies with:
	- Capacity constraints from `p_nom`
	- Availability from `p_max_pu` (aggregated to CapacityFactor)
	- Variable costs from `marginal_cost` (start-up costs omitted)
4. Network Impedance and Voltage Constraints
	- Line resistance `r` and reactance `x`
	- Line conductance `g` and susceptance `b`
	- Line type parameters (r_per_length, x_per_length)
	- Transformer impedance parameters
	- Voltage magnitude limits `v_mag_pu_min`, `v_mag_pu_max`
	- Voltage nominal values `v_nom`
	**Translation Approach**: All impedance and voltage parameters are **ignored**. OSeMOSYS represents energy transfer via TradeRoute (if multi-region) with simple capacity limits, not impedance-based flow equations.
5. Transformer Operational Parameters
	- Tap ratio `tap_ratio`
	- Tap position `tap_position`
	- Phase shift `phase_shift`
	- Transformer type specifications
	**Translation Approach**: Transformers are **ignored** unless they define regional boundaries (similar to Lines). Intra-regional transformers are abstracted away.
6. ShuntImpedance Components
	- Shunt conductance `g`
	- Shunt susceptance `b`
	- Reactive power compensation devices
	**Translation Approach**: ShuntImpedance components are **completely ignored**.
7. Flexible Snapshot Weightings
	- Multi-category snapshot weightings (objective = weight in cost minimization objective, store = weight for storage state-of-charge dynamics, generator = weight for global constraints and energy totals)
	- Snapshot-specific weightings for different cost/balance equations
	- Non-uniform temporal weighting strategies
	**Translation Approach**: PyPSA snapshot weightings translate to OSeMOSYS YearSplit using **objective weightings** as primary. Store and generator weightings are **not separately represented**, potentially causing discrepancies if PyPSA model uses different weighting categories strategically.
8. Carrier-Specific Advanced Features
	- Carrier emissions factors `co2_emissions` (partially translated in future)
	- Carrier colors and visual attributes (metadata only)
	- Carrier-specific growth limits `max_growth` (partially translatable)
	- Carrier-specific capacity limits `max_capacity` (partially translatable)
	**Translation Approach**:
	- **Emissions**: Planned for Phase 2 enhancement (map to EmissionActivityRatio, EmissionsPenalty)
	- **Growth/capacity limits**: Translatable to OSeMOSYS TotalAnnualMaxCapacity or global constraints if needed
	- **Visual metadata**: Ignored (not relevant to optimization)
9. GlobalConstraint Advanced Features
	- Stochastic programming constraints
	- Multi-scenario constraints
	- Custom user-defined constraint functions
	**Translation Approach**: Common constraints (emissions cap, RE target, capacity limit) are translatable to OSeMOSYS equivalents. Exotic or stochastic constraints require **manual OSeMOSYS model modification** post-translation.
10. Investment Period Weightings
	- Investment period objective weightings
	- Investment period "years" elapsed time specification
	**Translation Approach**: PyPSA investment periods map to OSeMOSYS YEAR set entries. Investment weightings are **approximated** via DiscountRate and year spacing, but full control over investment period objective weights is not preserved.

# Translation Status
This section documents which PyPSA components and parameters are handled in the **pyoscomp** translation framework for generating OSeMOSYS-compatible datasets. Translation follows the phased Implementation Plan with foundational research complete and translation layer implementation in progress.
### Status Legend
- ✓ **Implemented**: Translation logic fully operational
- ○ **Partial**: Research complete, implementation in progress
- ✗ **Not Implemented**: Planned for future phases
### Bus Component Translation
| PyPSA Attribute    | pyoscomp Translation Method                                | OSeMOSYS Mapping                            | Status |
| ------------------ | ---------------------------------------------------------- | ------------------------------------------- | ------ |
| `name`             | `translation/pypsa_translator.py`<br>- `translate_buses()` | REGION.csv entry                            | ✗      |
| `carrier`          | Translation logic                                          | Fuel / commodity categorization             | ✗      |
| x, y (coordinates) | Geographic metadata                                        | Not directly translated (metadata only)     | ✗      |
| `v_nom`            | Network parameter                                          | Not translated (OSeMOSYS abstracts voltage) | ✗      |
**Translation Approach**: Each PyPSA bus maps to one OSeMOSYS REGION. Carrier attribute determines fuel/commodity generation strategy (electricity, heat, hydrogen, etc.). Geographic coordinates preserved as metadata but not used in OSeMOSYS optimization.
### Generator Component Translation
| PyPSA Attribute          | pyoscomp Translation Method                                     | OSeMOSYS Mapping                                               | Status |
| ------------------------ | --------------------------------------------------------------- | -------------------------------------------------------------- | ------ |
| `p_nom` (fixed)          | `translation/pypsa_translator.py`<br>- `translate_generators()` | ResidualCapacity.csv                                           | ✗      |
| `p_nom` (extendable)     | Translation logic                                               | Capacity optimization variables                                | ✗      |
| `p_max_pu` (time series) | Timeslice aggregation                                           | CapacityFactor.csv (by TIMESLICE)                              | ✗      |
| `efficiency`             | Performance translation                                         | InputActivityRatio.csv (1/efficiency), OutputActivityRatio.csv | ✗      |
| `capital_cost` (€/MW)    | Economics translation                                           | CapitalCost.csv (€/GW, multiply by 1000)                       | ✗      |
| `marginal_cost` (€/MWh)  | Economics translation                                           | VariableCost.csv (€/PJ, unit conversion)                       | ✗      |
| `carrier`                | Carrier mapping                                                 | Input / output fuel determination                              | ✗      |
| `bus`                    | Topology linkage                                                | Region assignment                                              | ✗      |
**Translation Challenges**:
- **`p_max_pu` aggregation**: PyPSA snapshot time series → OSeMOSYS timeslice representative values (weighted average or percentile-based)
- **Unit conversions**: MW→GW (×1000), €/MWh→€/PJ (×3.6 then adjust for activity units)
- **CapacityToActivityUnit**: Critical conversion factor (31.536 for PJ/GW/year or 8760 for GWh/GW/year)
### Load Component Translation
| PyPSA Attribute       | pyoscomp Translation Method                               | OSeMOSYS Mapping                                        | Status |
| --------------------- | --------------------------------------------------------- | ------------------------------------------------------- | ------ |
| `p_set` (time series) | `translator/pypsa_translator.py`<br>- `translate_loads()` | SpecifiedAnnualDemand.csv<br>SpecifiedDemandProfile.csv | ✗      |
| `bus`                 | Topology linkage                                          | Region assignment for demand                            | ✗      |
| `carrier`             | Fuel mapping                                              | Demand fuel specification                               | ✗      |
**Translation Approach**:
1. Calculate annual total: `SpecifiedAnnualDemand = sum(p_set × snapshot_weightings)`
2. Generate timeslice profile: `SpecifiedDemandProfile[l] = (demand in timeslice l) / total annual`
3. Aggregate snapshots to timeslices using predefined timeslice structure
### StorageUnit Component Translation
| PyPSA Attribute          | pyoscomp Translation Method                                        | OSeMOSYS Mapping                                           | Status |
| ------------------------ | ------------------------------------------------------------------ | ---------------------------------------------------------- | ------ |
| `p_nom` (power capacity) | `translation/pypsa_translator.py`<br>- `translate_storage_units()` | STORAGE capacity = `p_nom` x `max_hours` (energy)          | ✗      |
| `max_hours` (E/P ratio)  | Decomposition logic                                                | Energy capacity calculation                                | ✗      |
| `efficency_store`        | Storage charge tech                                                | InputActivityRatio for charge TECHNOLOGY                   | ✗      |
| `efficiency_dispatch`    | Storage discharge tech                                             | OutputActivityRatio for discharge TECHNOLOGY               | ✗      |
| `standing_loss`          | Storage loss approximation                                         | MinStorageCharge approximation or ignored                  | ✗      |
| `capital_cost`           | Economics translation                                              | Split between STORAGE and charge/discharge TECHNOLOGY(ies) | ✗      |
**Translation Strategy** (Decomposition):
1. Create **STORAGE facility** with capacity `e_nom = p_nom × max_hours`
2. Create **charge TECHNOLOGY** with `TechnologyToStorage` linkage, efficiency from `efficiency_store`
3. Create **discharge TECHNOLOGY** with `TechnologyFromStorage` linkage, efficiency from `efficiency_dispatch`
4. Couple charge/discharge capacity to storage power rating

**Critical Note**: Single PyPSA StorageUnit becomes **three OSeMOSYS entities**, requiring careful tracking and naming conventions (e.g., `BATTERY_01`, `BATTERY_01_CHARGE`, `BATTERY_01_DISCHARGE`).
### Store Component Translation
| PyPSA Attribute           | pyoscomp Translation Method                                 | OSeMOSYS Mapping                         | Status |
| ------------------------- | ----------------------------------------------------------- | ---------------------------------------- | ------ |
| `e_nom` (energy capacity) | `translation/pypsa_translator.py`<br>- `translate_stores()` | STORAGE.csv entry                        | ✗      |
| `e_min_pu`, `e_max_pu`    | Storage constraints                                         | MinStorageCarge, storage capacity limits | ✗      |
| `e_initial`               | Initial state                                               | StorageLevelStart.csv                    | ✗      |
| `e_cyclic`                | Cyclic constraint                                           | End-of-year = start-of-year constraint   | ✗      |
**Translation Strategy**: Store requires associated Link components for charging/discharging. Translation identifies Links connected to Store and decomposes into charge/discharge TECHNOLOGY entities with `TechnologyToStorage` / `TechnologyFromStorage` linkages.
### Link Component Translation
| PyPSA Attribute                  | pyoscomp Translation Method                                | OSeMOSYS Mapping                                                         | Status |
| -------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------ | ------ |
| `p_nom` (capacity)               | `translation/pypsa_translator.py`<br>- `translate_links()` | TECHNOLOGY capacity or trade route capacity                              | ✗      |
| `efficiency`, `efficiency2`, ... | Multi-port decomposition                                   | MODE_OF_OPERATION with InputActivityRatio / OutputActivityRatio per port | ✗      |
| `bus0`, `bus1`, `bus2`, ...      | Port configuration                                         | Region fuel flow definitions per mode                                    | ✗      |
| `capital_cost`, `marginal_cost`  | Economics translation                                      | CapitalCost, VariableCost for TECHNOLOGY                                 | ✗      |
**Translation Strategy** (Multi-Port Links):
1. Create single TECHNOLOGY for Link
2. Each port combination → separate MODE_OF_OPERATION entry
3. Input ports (bus0, negative efficiency) → InputActivityRatio per mode
4. Output ports (bus1+, positive efficiency) → OutputActivityRatio per mode
5. Enables sector coupling (e.g., electrolyzer: electricity input, H2 output, heat output)

**Example**: 3-port Link (electricity→hydrogen+heat) becomes TECHNOLOGY with MODE1 having:
- InputActivityRatio[ELEC] = 1.0 / efficiency
- OutputActivityRatio[H2] = efficiency × port1_fraction
- OutputActivityRatio[HEAT] = efficiency2 × port2_fraction
### Line Component Translation
| PyPSA Attribute     | pyoscomp Translation Method | OSeMOSYS Mapping                                       | Status |
| ------------------- | --------------------------- | ------------------------------------------------------ | ------ |
| `bus0`, `bus1`, ... | Regional mapping            | TradeRoute if buses map to distinct REGIONs            | ✗      |
| `s_nom` (capacity)  | Trade capacity              | Trade capacity limit (if applicable)                   | ✗      |
| r, x (impedance)    | Network physics             | Not translated (OSeMOSYS lacks network flow equations) | ✗      |
**Translation Limitation**: Lines represent **AC/DC power flow with network physics** (Kirchhoff's laws, impedance). OSeMOSYS uses **commodity flow model** without network equations. Translation is only meaningful if:
1. PyPSA buses aggregate to distinct OSeMOSYS regions
2. Lines between regions map to TradeRoute
3. User accepts loss of network flow detail (impedance, voltage, reactive power)

**Default Approach**: Single-region models ignore Lines (intra-regional network abstracted away). Multi-region models require user specification of bus→region mapping.
### Transformer Component Translation
| PyPSA Attribute    | pyoscomp Translation Method | OSeMOSYS Mapping                        | Status |
| ------------------ | --------------------------- | --------------------------------------- | ------ |
| `bus0`, `bus1`     | Voltage level mapping       | Not directly translated                 | ✗      |
| `s_nom` (capacity) | Capacity limits             | Not translated unless regional boundary | ✗      |
**Translation Limitation**: Transformers model **voltage transformation between buses**. OSeMOSYS abstracts voltage levels entirely. Transformers are ignored in translation unless they define regional boundaries (similar to Line treatment).
### Economics Integration
| PyPSA Attribute                         | pyoscomp Translation Method                                   | OSeMOSYS Mapping                        | Status |
| --------------------------------------- | ------------------------------------------------------------- | --------------------------------------- | ------ |
| `capital_cost` (various components)     | `translation/pypsa_translate.py`<br>- `translate_economics()` | CapitalCost.csv (with unit conversion)  | ✗      |
| `marginal_cost` (Generator, Link, etc.) | Economics translation                                         | VariableCost.csv (with unit conversion) | ✗      |
| Discount rate (not explicit in PyPSA)   | User-specified parameter                                      | DiscountRate.csv (required by OSeMOSYS) | ✗      |
**Translation Challenges**:
- **Unit conversions**: PyPSA uses €/MW for capital, €/MWh for variable. OSeMOSYS uses consistent capacity/activity units (€/GW, €/PJ). Conversion factors must align with CapacityToActivityUnit.
- **Discount rate**: PyPSA doesn't require explicit discount rate (can be applied in post-processing). OSeMOSYS requires DiscountRate.csv for NPV calculations. Translation must accept user-specified value or apply default (e.g., 5%).
- **Fixed O&M**: PyPSA doesn't separate capital from fixed O&M as clearly as OSeMOSYS. May require user specification or standard assumptions (e.g., 2-4% of capital cost annually).
### Time Series Aggregation
| PyPSA Attribute                                     | pyoscomp Translation Method                                     | OSeMOSYS Mapping                                                 | Status |
| --------------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------------------------- | ------ |
| `snapshots` (flexible time steps)                   | `translation/pypsa_translator.py`<br>- `aggregate_timeseries()` | TIMESLICE (predefined structure)                                 | ✗      |
| `snapshot_weightings` (objective, store, generator) | Aggregation logic                                               | YearSplit fractions (Conversionlh x Conversionld x Conversionls) | ✗      |
| Time series (`p_max_pu`, `p_set`, etc.)             | Statistical aggregation (mean, percentile)                      | Timeslice representative values                                  | ✗      |

**Translation Approach**:
1. **Predefine timeslice structure**: User specifies SEASON × DAYTYPE × DAILYTIMEBRACKET (e.g., 4×2×3 = 24 timeslices)
2. **Map snapshots to timeslices**: Assign each PyPSA snapshot to one OSeMOSYS timeslice based on temporal attributes
3. **Aggregate time series**: For each timeslice, calculate representative value:
- **CapacityFactor**: Weighted mean or conservative percentile (e.g., P10 for reliability)
- **Demand profile**: Mean demand in timeslice / annual total
- **Snapshot weightings**: Sum weightings in timeslice → YearSplit fraction

**Key Decision**: Timeslice aggregation method (mean vs percentile) significantly affects results. Conservative approach uses low percentile for generation availability, high percentile for demand to ensure adequacy.
### Carrier Mapping
| PyPSA Attribute      | pyoscomp Translation Method | OSeMOSYS Mapping                    | Status |
| -------------------- | --------------------------- | ----------------------------------- | ------ |
| AC, DC (electricity) | Carrier translation         | ELEC or ELEC_HV / ELEC_MV / ELEC_LV | ✗      |
| heat, urban_heat     | Carrier translation         | HEAT_DISTRICT or HEAT_LOW_TEMP      | ✗      |
| H2, hydrogen         | Carrier translation         | H2_COMPRESSED or H2                 | ✗      |
| gas, natural gas     | Carrier translation         | GAS_NAT                             | ✗      |
| oil, petroleum       | Carrier translation         | OIL_CRUDE or petroleum products     | ✗      |
| biomass              | Carrier translation         | BIOMASS_SOLID or BIOMASS_GAS        | ✗      |
**Translation Strategy**: PyPSA carriers define energy types flowing through network. Translation creates corresponding OSeMOSYS FUEL entries. Mapping can be:
- **1:1**: Simple carrier (e.g., `H2` → `H2`)
- **1:many**: Carrier with voltage levels (e.g., `AC` → `ELEC_HV`, `ELEC_MV`, `ELEC_LV`)
- **User-specified**: Custom mapping for specific modeling needs
### Implementation Notes

**Phase 4 (In Progress)**: Translation layer orchestrates component usage to convert PyPSA Network objects to OSeMOSYS CSV datasets. Key tasks:
- Bus→REGION, Carrier→FUEL, timeslice aggregation framework
- Generator, Load, StorageUnit, Link translation with decomposition logic
- Economics integration with unit conversion validation

**Critical Path Items**:
1. **CapacityToActivityUnit**: Conversion factor must be consistent across all translations (31.536 for PJ/GW/year or 8760 for GWh/GW/year)
2. **Timeslice structure**: User must specify before translation (cannot dynamically adjust OSeMOSYS timeslices)
3. **Storage decomposition naming**: StorageUnit generates 3 entities requiring clear naming convention
4. **Multi-port Link modes**: MODE_OF_OPERATION generation must handle arbitrary port counts

**Validation Strategy**: Translation outputs validated against:
- Energy balance: PyPSA annual energy totals ≈ OSeMOSYS annual activity × CapacityToActivityUnit
- Capacity limits: PyPSA p_nom constraints ≈ OSeMOSYS TotalCapacityAnnual
- Cost totals: PyPSA objective function ≈ OSeMOSYS TotalDiscountedCost (within unit conversion tolerance)