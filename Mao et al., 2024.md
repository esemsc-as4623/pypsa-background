#storage #deeplearning

https://www.sciencedirect.com/science/article/pii/S0378779624003365?pes=vor&utm_source=clarivate&getft_integrator=clarivate

- three types of energy storage systems
    - mechanical
    - electrochemical
    - electromagnetic
- each type has unique characteristics
    - power
    - installed capacity
    - response time
    - life span
    - cost

- multi-timescale dynamics of energy storage system differs from traditional synchronous generators
- results in challenges for accurate and efficient simulation of power system with multiple energy storage systems

physical-based models
- white-box
- chemistry, thermodynamics, kinetics
- PDEs
- limitation: increase solution complexity as model order and number of unknowns increases

equivalent circuit models
- grey-box
- series of circuit elements to match physical phenomenon inside energy storage unit and describe internal measurable response (e.g. voltage, current)
- state estimation
- limitation: internal properties are constant

empirical / data-driven models
- black-box
- experimental data to interpolate b/w local linear models while capturing non-linear effects
- identify relationships b/w internal variables through deep learning w/o prior knowledge

- need to observe behavior of storage at different time scales
- diverse technological focuses of storage systems, each with its own features