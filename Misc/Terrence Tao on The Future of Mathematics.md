30th Dec 2025 (https://www.youtube.com/watch?v=4ykbHwZQ8iU)

something missing in the way we do math
- find the cool idea to unlock
- lots of literature review
- all inputs are different
- hand tweak arguments to make them applicable
- slog of computing
- helpful to build intuition

machine-assisted proof
- software packages, solvers, LLMs, LEAN
- suddenly things were possible
- formalization projects of the 20th century took decades
- liquid tensor experiment formalized in 18 months
- already a speedup due to using technology

future of mathematics = machine-assisted proofs
- learned LEAN (took about a month)
- rigorous
- like playing a computer game

LLMs can now formalize individual steps
- "I had not learned to think clearly about mathematics" - interviewer
- how did cognition change?
	- what is the cleanest way to define things
	- set up trivial memos with LEAN
	- if you don't define things correctly, it takes longer to formalize
	- gives perspective to clean up writing
- proofs need not be linear
	- here are 10 facts, find some combination to get to a conclusion
	- see more clearly the inputs and outputs
	- can be harder to check but easier to follow
- the process of getting feedback from key lemmas adds rigour
	- pre-rigour: don't know what the proof is but you know what works and doesn't
	- rigour: do things correctly by the book, but often lose intuition but lose bad intuition
	- post-rigour: switch back and forth, argue informally but safely because you know it
- sometimes when you state a theorem you add too many assumptions
	- LEAN allows you to realize you didn't need some constraints
	- formalization cleans up what the assumptions and outputs are
	- this is just good software engineering practice
	- e.g. Djikstra
- what is interesting about formalization?
	- story 1: formalizing PFR conjecture (combinatorics)
		- 3 weeks of effort: c = 12
		- someone on arXiv changed some things to make c = 11
		- using LEAN, change c = 11
		- 5 statements highlighted red as they are no longer valid
		- then you can fix them individually
	- story 2: someone else formalizing using LEAN
		- from one line, you get enough context
		- this can give sufficient information to diagnose problems
		- atomic discussions about specific lines of code without needing to know the rest of the code
		- could only have been done with "the perfect collaborator" before
			- same definitions and understanding
			- subject area expert
			- good intuition
- how has instinct for collaboration evolved?
	- there is more math that I want to do than I can do alone
	- started a blog because people were reading and answering
	- polymath projects to crowdsource math
	- more people = more chance connections
	- always bottlenecked to the checker for coherency
	- paradigm didn't scale but it enabled broad projects with chance connections
	- no organizational structure in blogs / wikis
	- now moved on to github
	- before, everyone had to know everything (math, LaTeX, proofs)
	- now, you just need enough people to cover all the functions
	- true economies of scale with specialization
	- there will still be traditional hand-crafted math which will be valuable

definition of mathematics
- will broaden
- good project managers with enough math to coordinate
- some good at formalizing and using AI tools
- people can join and leave with fluidity
- everyone being involved in everything is optional
- math research is intimidating

there is a great demand for citizen science / math
- the Erdos Problem community
- modularize different aspects of the problem
- comment on someone else's proofs
- do some numerics
- there are people who want to do research-level math

formalizing bounds for analytic number theory
- automation is a complementary tool for human thought
- analytic number theory has neat ideas and tools
	- conversion of statements to things that are applicable to your problem
	- need to unpack the proof and build to your needs
	- harmonize notations; add some cheats
	- replace constants with c, the price you pay is that results are not explicit
	- more tedious to work out the constants, these papers are unpleasant to read
	- this is a prime target for automation
- set up a pipeline that takes the explicit papers
	- match together some slightly incompatible tools
	- get parameters to match up
	- can be done at scale
	- find the optimal way to chain things together
- new way to look at the field
	- e.g. new proof of bounds
	- automated like an excel spreadsheet
	- living dynamic state of the field
	- no need to rewrite existing papers to add new information
- it is interoperable
- with LEAN it can hopefully be bug free

there is precendent to using computers in mathematics (data-driven)
- formalization is not the same but it is computer-assisted
- because of the drudgery, we change the way we do math
- when it starts to look messy, we try to change the way to do it
- this is to avoid potholes
- when these tools are perfected, we will just plough through
- there are some missed opportunities because we subconsciously change direction

trust issue
- who are reliable authors and who are not (implicit information)
- need to be in the network to be able to work in the field
- trust guarantee can open it up to other people
- especially if guaranteed by LEAN
- analysis is closer to first principles so this is less of an issue