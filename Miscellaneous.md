## Repo Management
- Choose A License: www.choosealicense.com, https://creativecommons.org/choose/
## Learning
- Imperial Research Computing & Data Science (https://github.com/ImperialCollegeLondon/RCDS-materials?tab=readme-ov-file)
	- Command line for scientific computing: https://github.com/ImperialCollegeLondon/RCDS-comm-line
	- SWE for Researchers: https://imperialcollegelondon.github.io/grad_school_software_engineering_course/
	- OOP in Python: https://github.com/ImperialCollegeLondon/RCDS-object-oriented-python
	- Profiling and optimization in Python: https://github.com/ImperialCollegeLondon/RCDS-profiling-and-optimisation-in-python
	- Intro to containers: https://github.com/ImperialCollegeLondon/RCDS-intro-to-containers
- TU Berlin Data Science for ESM: https://fneum.github.io/data-science-for-esm/intro.html
- Operations Research: https://ap-rg.eu/courses/operations-research-linma-2491/
- Mixed Integer Programming: https://www.youtube.com/@mixedintegerprogramming/videos [[Mixed Integer Programming Notes]]
## Research
- [[Information Retrieval]]
- Summarization Prompt:
```markdown

You are an expert researcher who reads and summarises scientific, peer-reviewed articles for domain experts and enthusiasts.\ \ Below is an excellent example of a summary you have written and attached is the original published article that it is about.\ \ Write a summary for a new paper using the same style as your example. Be factually correct. 

<example> 

🌍 The Challenge: 

The ocean is the beating heart of the Earth system. But observing it is incredibly difficult: 

🚢 Floats and drifters move with currents (Lagrangian observations) 

🛰️ Satellites provide only thin, track-based snapshots (often less than 1% coverage per day) 

🌊 Subsurface regions are nearly blind spots 

This extreme sparsity and irregularity means we often lack reliable initial conditions for weather, climate, and ocean forecasts. Traditional assimilation methods (e.g., ensemble Kalman filters, variational approaches) are powerful but computationally very expensive and often degrade in extremely sparse regimes. 

🤖 Our Solution: 

We built a generative, data-driven assimilation pipeline that works without requiring forward models: 

🔹 Fourier Neural Operator (FNO) → learns physics-aware, coarse priors from sparse input 

🔹 Denoising Diffusion Probabilistic Model (DDPM) → refines those priors into realistic, high-resolution reconstructions 

Together, this FNO+DDPM framework can: 

- Recover eddies, filaments, and fine-scale turbulence 

- Preserve spectral consistency (energy across spatial scales) 

- Handle 99–99.9% sparsity in both synthetic and real satellite altimetry 


📊 Results That Excite Us: 

✅ On synthetic turbulence → Reconstructed sharp vorticity filaments missed by UNET/FNO baselines 

✅ On regional reanalysis → Captured mesoscale & sub-mesoscale structures (critical for eddy-rich regions) 

✅ On real altimetry → Reconstructed full-resolution SSH fields from just 0.1-1% observations 

Importantly, we show that common ML metrics (RMSE, SSIM) is often misleading in terms of the physics it captures — our models shine in spectral energy and physical diagnostics (strain rate, vorticity). 


🌟 Why It Matters: 

This approach offers a fast, one-shot alternative to computationally heavy data assimilation: 

⚡ Efficient: no tangent-linear/adjoint solvers, no massive ensembles 

🌡️ forecast-ready: can feed initial conditions into ML-based/physics-based ocean models 

🌊 Extensible: applicable to subsurface reconstructions (thermocline, salinity) and global ocean data from SWOT/Jason missions 

💻 Code & models: GitHub – https://lnkd.in/gq7kG3Ru 

🔗 Full paper: https://lnkd.in/gMZFRgiT 

</example>
```