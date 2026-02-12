2026-02-11

Deep Learning Institute (https://www.nvidia.com/en-gb/training/)
- teaching kits
- self-paced learning

Academic Grant Program
- reviewed by NVIDIA researchers

3-computer diagram: train --> simulate --> deploy (closed loop)
- Nvidia is the enabler of the physical AI ecosystem
- software + hardware design
- blueprints and frameworks for creating synthetic data and validation (open source)

Simulate: Omniverse, Isaac Sim, Isaac Lab
Train: DGX
Deploy: Holoscan

Isaac Lab 3.0 Architecture
- frontend
	- PyTorch
	- **Warp with CUDA Graphs**
- dependencies
	- PhysX
	- **Newton**
	- RTX SDK
	- Warp Renderer
	- Newton Visualizer
	- Isaac Teleop
	- Interaction Kit GUI
- intermediaries
	- physics simulation
	- rendering
	- visualization
	- Isaac Sim / Omniverse Kit

Goal:
- multiple frontends
- multiple simulation engines
- multiple rendering engines
- easy install

Newton is a simulation framework (open source)
- built by Disney, DeepMind, Nvidia, Linux Foundation
- goals:
	- fidelity
	- performance
	- modularity
- design principles
	- separate physical model from numerical method (to be able to switch solvers)
	- flat data > OOP (efficiency and memory assignment)
	- avoid hidden state (more control over memory for computations, e.g. backpropagation)
	- take what you need at each layer (function, kernel, solver)
- written on top of Nvidia Warp

Nvidia WARP
- CUDA speed with Python simplicity
- differentiable programming
- rich vector math library with specific data structures (implemented in C)
```python
import warp as wp

@wp.kernel
def integrate(x: wp.array(dtype=wp.vec3),
			  v: wp.array(dtype=wp.vec3),
			  f: wp.array(dtype=wp.vec3),
			  m: wp.array(dtype=float)):
	# thread ID
	tid = wp.tid()
	# simple implicit Euler step
	v[tid] = v[tid] + (f[tid] * m[tid] + wp.vec3(0.0, -9.8, 0.0))*dt
	x[tid] = x[tid] + v[tid]*dt
	
# kernel launch
wp.launch(integrate, dim=1024, inputs=[x, v, f, _], device="cuda:0")
```

Newton
- `newton.Model` = system description
- flat Python struct of 1D, 2D WARP arrays
- fundamentally store data grouped by frequencies
- support for generalized and maximal coordinates
- flexible with Python's dynamic typing
```python
class Model:
	"""
	Holds the definition of the simulation model.
	
	Holds non-time varying description of the system, i.e.:
	- all geometry
	- all constraints
	- all parameters
	used to describe the simulation.
	
	Attributes:
		body_com : array(dtype=wp.vec3)
		body_inertia : array(dtype=wp.mat33)
		body_inv_inertia : array(dtype=wp.mat33)
		body_mass : array(dtype=float)
		
		joint_type : array(dtype=int)
		joint_parent : array(dtype=int)
		joint_child : array(dtype=int)
		joint_X_p : array(dtype=wp.transform)
		joint_X_c : array(dtype=wp.transform)
		joint_axis : array(dtype=wp.vec3)
		
		spring_indices : array(dtype=int)
		spring_rest_length : array(dtype=float)
		spring_stiffness : array(dtype=float)
		spring_damping : array(dtype=float)
		spring_control : array(dtype=float)
	"""
```

Example simulation pipeline
- model builder --> update `model`, `state`, `control`
- `model`, `state`, `control` --> input into solver --> update `state`
- additionally: collision pipeline to generate `contacts`
Solvers
- canonical, RBD, deformable
- implemented with greater degrees of freedom than JAX
Newton simulation loop (write your own)
- `model`, `state` --> `model.collide(state)` --> collision pipeline --> `contacts` --> solver
- split between contact detection and time stepping
- physics solver need not know about geometry
- contact detection finds overlapping points and detects points where reactions should act to keep the objects properly separated
- created in speculation to model forces correctly (i.e. just before actual contact)
- challenges
	- shape variety
	- number of contact points
	- pairwise simulations grow quadratically in complexity
- solutions
	- create representative contacts
	- histogram binning of contact normal
	- search largest polygon of contacts
	- BUT this relies on problem heuristics which are very specific
```bash
# Clone the repository
git clone git@github.com:newton-physics/newton.git
cd newton

# set up the uv environment for running Newton examples
uv sync --extra examples

# run an example
uv run -m newton.examples basic_pendulum
```

Nvidia Cosmos = world foundation model development platform
- NNs that are able to understand and model the dynamics of the physical world
- multi-modal + adherence to consistency
- sensor + text tokens --> physical foundation model --> action tokens
- curate representative data, train, evaluate, ...
- Cosmos Cookbooks (https://nvidia-cosmos.github.io/cosmos-cookbook/index.html)

Robotics in healthcare
1. robotic surgery = subtask automation
2. telesurgery = mirroring action (high quality concurrent data collection)
3. robotic scans = imitation learning
- Markov decision process
- state machine for data collection
- generate paired surgical video
- Neural tokenizers (e.g. encoder to reduce data requirements and latency)

Gaze is a proxy for attention?
- precedes action
- action-grammars
- memory component
	- short-term = current attention
	- long-term = previously paid attention
	- prior attention can be actionable later
- affordance-action predictor
	- affordance = predict the situation based on context clues

Sample efficient algorithms
- action space, architecture, pre-training
- continuous control is sample inefficient and unstable in training
- discrete action space using a value function (BUT this loses information)
- solution: do this repeatedly to zoom into action space until you have the final action (neo-continuous)
- coarse-to-fine reinforcement learning

In-context imitation learning = immediate learning following a demonstration

Quality diversity optimization
- creative diversity of solving a task
- unbounded creative skill discovery
- framework:
	- repertoire of representations of actions
	- select and mutate from repertoire
	- generate sensor data (synthetic)
	- sample from data
	- dimensionality reduction to add to repertoire
	- update master training data (created from repertoire)