Claude, Codex, GPT5-Pro

currently cognitively lacking
- not continually learning
- not multimodal

what's the timeline?
- problems are surmountable
- might take about a decade?
- some seismic shifts
	- NNs were a niche research domain, trained on very specific tasks
	- RL agents that perceive the world (Atari)
- worked on keyboard-mouse agent that interacts with the digital world
- people try to get the full thing too early
	- we want something like an accountant that interacts with a world - so how does a game fit in?

RL
- don't think we use RL for intelligent tasks
	- there is some crazy compression and learning algorithm encoded in our DNA
- more useful for motor tasks
- practical POV: let's build something useful
- we try many things in parallel and check what worked
	- RL upweights every step that led to the right answer
	- the reward signal is broadcase to the entire trajectory
	- there is usually a more complicated process of review (no equivalent algorithm for this)
- InstructGPT shows that fine-tuning can change the style of the output with new labelled output
- in RL you can hill climb on the reward functions without the labelled output
- process-based supervision = reward at every step
	- difficult to assign credit
	- some labs try to do it with LLM-as-a-judge
	- LLM can find spurious patterns in the large model and try to exploit it
	- LLM will always find adversarial examples that the model generalizes as top quality
- can we meta-learn it to some extent?
	- some papers have cool ideas
	- to do it convincingly, we need it on the scale of a LLM lab

Pretraining VS In-Context Learning
- pretraining = next token predictor over the internet
	- picks up knowledge
	- becomes intelligent
	- maybe relies on the knowledge too much?
	- would they be better with less knowledge/memory?
	- remove knowledge and keep the cognitive core
- in-context learning = pattern completion within a token window
	- model learns to complete the pattern (inside the weights)
	- there is an adaptation inside the NN
	- small gradient descent-like updates in continual learning
	- knowledge is a hazy recollection of what happened in training (due to compression)
	- anything in the context window at test time is working memory (directly accessible)

Transformers
- general, train on any mode of input data
- this is like a piece of cortical tissue, rewire as needed
- there are other ancient nucleii in the brain that don't have analogous algorithms
- but are we trying to build a human brain?

- while we are awake, we build a context of our day
- when we sleep, there is a process of distillation into weights
- this is still missing; new chat = empty context, no tokens

10 years ago
- CNNs
- RNNs relatively new
- still training with gradient descient and backpropagation
- transformers were not around
- everything now is just bigger
- recreate Yann LeCunn's 1989 algorithm now; halve the training time

Nano Chat
- simplest repository to build a ChatGPT clone end-to-end
- entire pipeline
- reference but do not copy-paste
- two types of knowledge
	- high-level surface knowledge
	- deeper understanding when you try to build it
- models have too much memory from the typical ways of doing things
	- synchronize gradient updates between GPUs (can be done using a container)
	- wrote his own routine but they kept misunderstanding / pushing to use the container

Ways in which people interact with code
- write code from scratch; no LLMs (probably not the way anymore)
- write code with autocomplete (correct as needed)
- vibe-coding
	- agents work in specific settings
	- need to learn what they're good at and when to use them
	- good at boiler plate code
	- good at things that occur a lot in the internet
	- very willing to productionize
	- use deprecated APIs
	- bloat the codebase for no reason
	- not good at code that has never been written before
	- not good at custom design or style

Productivity Improvements in Coding
- before
	- linting
	- syntax highlighting
	- datatype checking
	- compilers
- now: agents are these loopy things that go off rails sometimes

Reflection
- human example: daydreaming / sleeping
- what could be missing:
	- in the pretraning stage, there is something that thinks through what it has learned and reconcile with what it already knows
	- there is no equivalent of any of this (all research)
	- can we synthetically generate something and train on it?
		- all the samples from a model are silently collapsed
		- they all occupy a small area of the possible space of thoughts about the content
		- you are not getting the richness, diversity, and entropy that we get from humans (statistically)
		- individual samples might look okay but the distribution is very narrow
	- humans also collapse over time
		- reasoning with the same thoughts
		- learning rates go down
		- children say things that are surprising because they have not overfit yet
- seek entropy in life (e.g. talking to different people)
- children are great at learning but bad at recollection
	- LLM pretraining is the opposite
	- humans are not good at memorization -- this is a feature
	- we are forced to find patterns / generalizable components
	- memorization content might be distracting to the LLM
	- less memory = look things up
	- only retain the cognitive glue / algorithms to learn

Model Collapse
- naive approaches
	- regularization for entropy
		- want more solutions / alternatives but don't want a new language to be created
		- tuning this parameter is not trivial
	- more diverse input samples
- most tasks we want don't demand diversity
	- difficult to evaluate (often actively penalized)
	- but we need it for other functions
- GPT-OSS-20B > GPT-5?
- training data is the internet, which is terrible
	- usually the data is garbage
	- WSJ articles and high-tier content is rare
	- we need intelligent models to refine the input data into the cognitive dataset
	- then train a smaller model on quality data
	- practically speaking you still want the model to retain some knowledge
	- basic curriculum but not esoteric knowledge
- 