#pypsa #osemosys 

Workflows:
1. Uniform, high resolution timeseries >> discrete, non-uniform `snapshots` (PyPSA)
2. Uniform, high resolution timeseries >> discrete, non-uniform `timeslices` (OSeMOSYS)
3. Discrete, non-uniform `snapshots` (PyPSA) >> discrete, non-uniform `timeslices` (OSeMOSYS)
4. Discrete non-uniform `timeslices` (OSeMOSYS) >> discrete, non-uniform `snapshots` (PyPSA)

**Workflow 4: `timeslice` >> `snapshots`**
`timeslice` = `season` x `daytype` x `dailytimebracket`
```python
for s in seasons:
	for d in datypes:
		for t in dailytimebrackets:
			snapshots.append(timeslice[s][d][t])
```

**Workflow 1: Discretization**
*Aggregating continuous data into the most significant segments*

Statistical methods
1. Ramer-Douglas-Peucker (RDP)
	- Iteratively removes points that contribute least to overall shape
	- Uses perpendicular distance from point to line segment
	- Preserves major features, removes redundant points
	- Single parameter: distance threshold
2. Visvalingam-Whyatt
	- Removes points based on effective area of triangle formed with neighbors
	- Progressively eliminates least significant points
	- Good for preserving visual appearance
3. Critical Point Sampling
	- Keep local maxima, minima, and inflection points
	- Can add midpoints between critical features
	- Simple and preserves important variations
4. Adaptive Rate-of-Change
	- Sample more densely where derivative is high
	- Sparse sampling in flat regions
	- Can use threshold on $|\frac{dy}{dt}|$ or second derivative
5. Sliding Window + Variance
	- Compute variance in sliding windows
	- Keep more points where variance is high
	- Simple statistical approach
6. Piecewise Linear Approximation (PLA)
	- Fit line segments with error tolerance
	- Keep only segment endpoints
	- Can use bottom-up, top-down, or sliding window methods
7. Perceptual Important Points (PIP)
	- Find point with maximum perpendicular distance from line connecting neighbors
	- Recursively subdivide
	- Similar to RDP but different recursion strategy
8. Time-Series Segmentation Algorithms
	- SWAB (Sliding Window and Bottom-up)
	- Online algorithms like SWAB-ED
	- Good for streaming data

ML methods
1. Change Point Detection / Regime Clustering
	- Detect when the timeseries transitions between different "regimes" or behavioral states.
		- **PELT (Pruned Exact Linear Time)** - Fast change point detection
		- **Bayesian Change Point Detection** - Probabilistic regime identification
		- **E-Divisive** - Non-parametric change detection
	- Good for: Wind data where generation shifts between high/low production regimes
- 2. K-Means on Sliding Windows
	- Create feature vectors from local windows and cluster them.
	- Extract features from sliding windows (mean, std, trend, etc.)
	- Cluster windows using K-means
	- Each cluster represents a "mode" of operation
	- Boundaries between clusters become discretization points
	- Good for: Identifying recurring patterns (calm periods, stormy periods, transitions)
3. Hidden Markov Models (HMM)
	- Model the timeseries as transitions between hidden states.
	- Fit HMM with N states to your timeseries
	- Each state represents a production regime
	- State transitions define discretization boundaries
	- Get both discrete states and transition probabilities
	- Good for: Modeling state-dependent behavior with temporal dependencies
4. Gaussian Mixture Models (GMM)
	- Soft clustering where each point has probabilities of belonging to multiple states.
	- Fit GMM to value distribution (possibly with temporal features)
	- Assign each point to most likely component
	- Adjacent points in same cluster form segments
5. Dynamic Time Warping (DTW) + Hierarchical Clustering
	- Cluster similar subsequences based on shape.
	- Good for: Finding recurring patterns regardless of timing
6. Symbolic Aggregate approXimation (SAX)
	- Convert timeseries to symbolic representation via clustering.
	- Divide into equal segments
	- Normalize each segment
	- Cluster segment values into discrete symbols
	- Results in discrete symbolic representation

**Workflow 2 & 3: Factorization**
*Aggregating continuous data into the most hierarchical groups*
Hierarchies: S `seasons`, D `daytypes`, T `dailytimebrackets`
$S*D*T = N$ `timeslices` 
User-specified `N` (following either discretization or any $N \leq len(x)$)
If `N` is prime and has no factors, $S = N, D = 1, T = 1$

1. Factorization Search with Scoring
	- Enumerate all valid (S, D, T) where S×D×T = N or S×D×T ≤ N (allow some merging), score each by quality metrics.
	- Finds optimal factorization, can balance multiple objectives 
	- Expensive (many factorizations to try), need good scoring function  
2. Temporal Blocks + Value-Based Refinement
	- Leverage natural temporal structure, then refine by value patterns.
	- Define seasons:
		- Fixed blocks (quarters, months)
		- Detected blocks (change point detection on smoothed series)
		- Spectral clustering on temporal features
	- Then, define daytypes:
	    - Within each season, cluster by value characteristics
	    - Options: quantiles, k-means on values, Gaussian mixture
	- Then, define timebrackets:
	    - Within each (season, daytype), cluster by hour-of-day
	    - Or: fixed hour blocks (morning/afternoon/evening)
	    - Or: adaptive based on load shape
	- Interpretable seasons, respects domain knowledge
3. Matrix Factorization / Tensor Decomposition
	- Reshape data into 3D array: [month × hour × day_of_year]
	- Apply tensor decomposition (CP, Tucker, or NMF)
	- Extract S, D, T factors from decomposition
	- Assign points based on factor loadings
	- Mathematically principled, finds latent structure  
	- Requires regular grid, hard to interpret factors
4. Recursive Binary Partitioning
	- Top-down splitting like a decision tree, choosing optimal split points at each level.
		```python
		  for i in range(S-1):
			  # find best split point that minimizes within_variance + temporal_penalty
			  # creates S season blocks
		  for s in seasons:
			  for j in range(D-1):
				  # find best split within season s (by value or pattern)
				  # creates D daytypes within season s
		  for s in seasons:
			  for d in s[daytypes]:
				  for k in range(T-1):
					  # find best split by hour or temporal order
					  # creates T timebrackets within (s, d)
	  ```
	- Interpretable, can enforce temporal contiguity  
	- Greedy (doesn't find global optimum)
5. Constrained Multi-level k-means
	- Apply k-means at each level, but constrain the feature space to guide the hierarchy.
	- Fast, flexible feature engineering, handles variable cluster sizes
	- Need to design features carefully, non-deterministic (sensitive to initialization)
6. Hierarchical Agglomerative Clustering
	- Start with N singleton clusters and merge based on distance metrics, then extract 3 levels from the dendrogram.
	- Compute pairwise distances between all points using hybrid metric:
	    - Temporal distance: $|time_i - time_j|$
	    - Value distance: $|value_i - value_j|$
	    - Combined: $α×temporal + β×value$
    - Build dendrogram via agglomerative clustering
	    - Cut at height $h_1$→ S clusters (seasons)
	    - Cut at height $h_2$ → S×D clusters (season-daytype pairs)
	    - Cut at height $h_3$ → S×D×T clusters (full timeslices)
	- Assign each point to its $(s, d, t)$ tuple
7. Graph-based Partitioning
	- Build a graph where nodes are points, edges weighted by similarity, then partition.
	- Build similarity graph:
		- Nodes: each point
		- Edges: connect temporally close OR value-similar points
		- Weights: $w_{ij} =$ temporal_similarity$(i,j) +$ value_similarity$(i,j)$
	- Apply graph partitioning:
		- Level 1: Spectral clustering with $k=S$
		- Level 2: Within each season, spectral clustering with $k=D$
		- Level 3: Within each $(s,d)$, spectral clustering with $k=T$

**Is it worthwhile to compare `len(snapshots) == len(timeslices)`?**
This requires methods to discretize the data, given `N`
1. Statistical Methods
	- Quantile-based
		- Divides data into bins with equal numbers of observations.
		- Each bin contains approximately n/k points where k is the number of groups.
		- Good for handling skewed distributions and ensures balanced representation.
	- Equal-width binning
		- Divides the value range into k equal intervals.
		- Simple but can create empty bins if data isn't uniformly distributed.
		- Works well when you care about the value space rather than point distribution.
	- Jenks Natural Breaks (Fisher-Jenks)
		- Optimizes the arrangement of values into k classes by minimizing within-class variance and maximizing between-class variance.
		- Particularly good for identifying natural groupings in the data. Commonly used in cartography and spatial analysis.
2. Clustering Methods
	- K-medoids (PAM - Partitioning Around Medoids)
		- Similar to k-means but uses actual data points as cluster centers, making it more robust to outliers.
		- Better than k-means when you want representative "typical" points from your time series.
	- Gaussian Mixture Models (GMM)
		- A probabilistic approach that models data as a mixture of k Gaussian distributions.
		- Provides soft clustering (probability of belonging to each cluster) and can capture elliptical cluster shapes better than k-means.
	- Spectral clustering
		- Uses eigenvalue decomposition of a similarity matrix.
		- Can find non-convex clusters and works well when clusters have complex shapes. 
		- Good for temporal data with periodic patterns.
3. Timeseries-specific Methods
	- Dynamic Programming Segmentation
		- Finds optimal k segments by minimizing total within-segment variance.
		- Guarantees the global optimum (unlike greedy methods) but is O(n²k) complexity. 
		- Very effective for piecewise-constant approximations.
	- Bottom-up segmentation
		- Starts with many segments and iteratively merges the most similar adjacent pairs until k segments remain.
		- Faster than DP but only locally optimal.
	- Top-down segmentation
		- Starts with one segment and recursively splits at the point of maximum error until k segments are created.
		- Related to greedy method but works in the opposite direction.
4. Domain-specific Methods
	- Value-based k-means
		- Cluster directly on values (not temporal features), then assign consecutive time periods to their nearest cluster representative.
	- Feature-based temporal clustering
		- Cluster based on extracted features (mean, variance, trend, periodicity) over sliding windows.
	- Constraint-based clustering
		- Add minimum/maximum segment duration constraints to ensure physically meaningful time blocks (e.g., each segment must be at least 1 hour or 1 week).
