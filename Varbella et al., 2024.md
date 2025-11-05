#gnn 

Dataset: https://figshare.com/articles/dataset/PowerGraph/22820534
Code: https://github.com/PowerGraph-Datasets

cascading outages are rare
- computationally intensive to model
- ML (e.g. GNNs) can offer potential for real-time solutions
- ML can replace traditional solvers in power flow simulations
- non-linear solvers for optimal power flow problems
- tested on synthetic power systems or benchmark IEEE power grids
- lack of generalizability across diverse failure sets
- lack of explainability hinders application of ML

PowerGraph
- dataset for (optimal) power flow problems using GNN supervised learning
- wide range of cascading failure scenarios
- real-world dataset with clear ground-truth explanations for GNN explainability
- node-level task: (optimal) power flow
	- each graph is a different grid loading condition
	- power flow analysis solves power flow equations, given power dispatched at each generator except for a balancing bus (slack bus)
	- optimal power flow problem determines optimal generator (dispatch) to operate the grid at minimum cost within physical constraints
	- predicted node-level features depend on type of bus
- graph-level task: cascading failure
	- N attributed graphs each representing a unique pre-outage operating condition and set of outages
	- outages result in removal of corresponding branches from adjacency matrix and edge features, altering graph topology
	- post-outage evolution is known for each initial state including demand not served (DNS), number of tripped lines (cascading edges)
	- each graph is assigned an output label
		- binary classification: DNS = 0 (stable) and DNS > 0 (unstable)
		- multi-class classification: 4 categories
		- regression: value of DNS (in MW)
	- explainability mask (ground truth)
		- cascading edges considered to be explanations for DNS
		- boolean vector, 1 for edges belonging to cascading stage, 0 o/w

architectures
- GCNConv
- GATConv
- GINEConv
- TransformerConv

explainability
- non-generative methods
	- heuristic occlusion
- gradient-based methods
	- saliency
	- integrated gradient
	- Grad-CAM
- perturbation-based methods
	- GNNExplainer
	- PGMExplainer
	- RCExplainer
	- SubgraphX
- generative methods
	- GSAT
	- D4Explainer
	- RCExplainer

