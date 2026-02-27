Speaker: Jean Kossaifi (ref: Nikola Kovachki)

Neural Operators: a framework for scalable scientific computing

weather is a hard problem
- we do not know the exact equations
- need to look at various resolutions to resolve different things
- SOTA: 25 km, 6 hours
- climate and weather predictions are limited by the computational power
- can AI achieve the increase in resolution at a reasonable timescale?

make decisions based on output of the model
- reason about the probability of specific events
- slightly perturb initial conditions
- map out the space of possibilities
- computation is even more prohibitive

neural operators
- build a surrogate model based on available data
- speed up the computation

operator learning = function learning ++
- systems governed by PDEs
- solutions are functions
- input = initial conditions
- output = functions
- instance-wise solving
- learn operator from any initial condition to solution function

any NN
- take an input vector (e.g. discretization of function)
- map to solution to be learned
- learn the function to do the mapping
- linear combination of inputs
- weights depends on the input points and output points
- for each input point, we learn a weight matrix
- with fixed discretization, need to add new weights and retrain everything to increase resolution

goals of neural operator
- need a way to scale the method
- resolution agnostic
- discretization conversion
	- we want things to stay consistent across resolutions
	- imagine training mostly at one resolution
	- if you have different resolutions, the learning should still be the same
- data efficient
- universal approximation
	- for NNs with enough capacity can approximate any continuous function between vector spaces
	- given an operator, we want to know that at least we can learn it

traditionally learn a function
now learn an operator (learn b/w function spaces)
- replace weight matrix by a kernel function
- linear layer becomes integral transform
- integrated w.r.t. input and output coordinates w/ bias (optional)
- query at any point in output function

importance to quadrature weights ($\Delta x_i$)
- if set incorrectly, solution will depend on the discretization
- if set correctly, will converge to true integral
- want something that is agnostic to the choice of the discretization

receptive field
- if receptive field depends on resolution, refining discretization, receptive field that collapses to a single point
- keep support fixed and accumulate information

look at continuum
- things stay coherent at the limit
- things stay coherent across resolutions

NN stacks learnable linear mappings
- neural operator generalizes this to functions
- encoder lifts to higher dimensional space
- neural operator layers
- decoder projects back to output space
- use integral linear operator to project on a space via non-kernel, then learn kernel

universal approximation theorem
- for any continuous operator, assuming large enough model, we can build a neural operator that approximates it

learning on a regular grid
- start with integral linear operator
- project to fixed basis space (e.g. Fourier basis functions)
- look at special case of linear operator: convolution
- perform convolution on Fourier domain
- learn directly the kernel in the Fourier space
- project input function in fixed basis function (fixed number of frequencies)
- do element-wise multiplication in Fourier domain
- inverse the Fourier transform

FFT = $O(n log(n))$

add residual connections to recover non-periodic and high frequency information
- train on low resolution
- inference at higher resolution
- NN cannot generalize but NO can

learning on a sphere
- just a transformer on ERA5 reanalysis data will not work on sphere
- Fourier transform on a sphere
- basis functions on a sphere (spherical harmonics)
- spherical convolution applied to each block
- residual connections to recover information
- concatenate some noise into the block to have uncertainty flow through the network
- input = state of the Earth at time $t$
	- take multiple variables at different altitudes
	- inject controlled randomness generate ensemble
- output = state of the Earth at time $t+1$
- conditioned on
	- initial state
	- auxiliary fields (e.g. solar azimuth)
	- latent noise field
- solution will depend on the noise
- use continuously ranked probability score
	- penalize distance to the ground truth + being too wide / between ensembles
	- if the model can choose any distribution, pick a "good" one
- get a model that outperforms traditional models

obtain many ensemble members in a single forward pass
- each one is physical
- use it to reason about events
- the more ensembles you can predict, the better probability you can get
- more true to for the tails of the distributions (extreme events)

learning on arbitrary geometries (graph)
- start with kernel linear integral
- integrate over local area (centered on query point w/ fixed radius)
- as you change the discretization, you will always look at the same spatial volume
- input samples (e.g. car geometries)
- encoding GNO
- apply FNO to latent geometry
- decoding GNO
- increase the size of the latent space

incorporating physics
- when we know the governing equations, use it as a loss
- data-driven operator learning phase
- plug in the physics and get actual solution
- fine-tune the model to an instance
- PINO has physics-informed fine-tuning

foundation models for scientific applications
- CoDA-NO (Co-Domain Attention Neural Operator)
- pre-train on a problem (e.g. Navier Stokes), apply to similar problem
- few shot supervised fine-tuning on new data
- variable specific positional encoding
- each variable is one channel and each has a learned specific encoding
- regular attention on FNO
- replace decoder with predictor branch that can be fine-tuned with new positional encodings for new variables
- learning a large range of PDEs

this is ONE example (Q: what are the others?)

scaling laws for NOs = same as for LLMs
- custom kernels to enable running at scale
- data and model parallelism
	- process less data on each GPU
	- shard model so each device processes part of the input
	- trade compute time for memory
- curriculum learning
	- train at low resolution
	- fine-tune or few epochs at high resolution
- tensor factorization
	- express weights in factorized form
	- reduce the number of parameters to learn (helps w/ regularization)
	- most memory is taken by the optimizer state
	- decompose (tensor grad); project gradients to low rank space and project back

always want to scale up the model, speed up training, reduce number of params

open source science and software
- torch harmonics
- makani
- neural operator
- Nvidia physics NeMO
- Nvidia earth-to-studio