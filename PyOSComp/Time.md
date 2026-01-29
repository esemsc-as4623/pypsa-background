#pypsa #osemosys 

## Translation

1. Discrete, non-uniform `snapshots` (PyPSA) >> discrete, non-uniform `timeslices` (OSeMOSYS)
2. Discrete non-uniform `timeslices` (OSeMOSYS) >> discrete, non-uniform `snapshots` (PyPSA)

**Workflow 1: `snapshots` >> `timeslice`**
`timeslice` = `season` x `daytype` x `dailytimebracket`
`snapshots`= `pd.DateTime()` or `pd.Index()`
mapping: one `snapshot` maps to at least one `timeslice` but one `timeslice` may be generated from at most one `snapshot` (many to one)
edge cases: may create some empty `timeslices`, particularly at the start and end, which may need to be intelligently filled (e.g. forward fill, default fill)
```python
daytypes, dailytimebrackets = {}, {}
snapshots = pd.to_datetime(snapshots).sort_values()

# data classes
@dataclass
class DayType:
    """
    Represents a day-of-year range (year-agnostic).
    Uses half-open interval [start, end).
    """
    month_start: int
    day_start: int
    month_end: int
    day_end: int
    name: str = field(default="")
    
@dataclass
class DailyTimeBracket:
    """
    Represents a time-of-day range (day- and year-agnostic).
    Uses half-open interval [start, end).
    """
    hour_start: dt_time
    hour_end: dt_time
    name: str = field(default="")
```
1. **Case 1**: `len(snapshots) < 2`
```python
# if one snapshot: one timeslice, representing whole year
if len(snapshots) < 2:
	years = sorted(snapshots.year.unique().tolist())
	daytypes = [
	        DayType(
	            month_start=1,
	            day_start=1,
	            month_end=12,
	            day_end=31,
	            name="ALL-YEAR"
	        )
	    ]
	dailytimebrackets = [
	        DailyTimeBracket(
	            hour_start=dt_time(0, 0, 0),
	            hour_end=dt_time(23, 59, 59, 999999),
	            name="ALL-DAY"
	        )
	    ]
	# timeslices = daytype x dailytimebracket
	# = {"FULL-YEAR, "FULL-DAY"}
```
2. create an array $\Delta(snapshots)$
```python
# create delta array
delta = snapshots[1:] - snapshots[:-1]
```
2. **Case 2:** annual resolution or coarser
```python
min_delta_years = (delta / np.timedelta64(1, 'Y')).min()
years = sorted(snapshots.year.unique().tolist())

# if annual or coarser
# 1 daytype per year, 1 dailytimebracket per day = 1 timeslice per year
if min_delta_years >= 1:
	daytypes.add(
		DayType(
			month_start=1,
			day_start=1,
			month_end=12,
			day_end=31,
			name="ALL-YEAR"
		)
	)
    dailytimebrackets.add(
		DailyTimeBracket(
			hour_start=dt_time(0, 0, 0),
			hour_end=dt_time(23, 59, 59, 999999),
			name="ALL-DAY"
		)
	)
    # timeslices = daytype x dailytimebracket
	# = {"ALL-YEAR, "ALL-DAY"}
```
3. **Case 3:** daily resolution or coarser
```python
min_delta_days = (delta / np.timedelta64(1, 'D')).min()

# if daily or coarser
# k daytypes per year, 1 dailytimebracket per day = k timeslices per year

# add year boundaries to discrete (month, day) tuples
unique_month_days = sorted(set((ts.month, ts.day) for ts in snapshots))
all_boundaries = [(1, 1)]  # start of year
all_boundaries.extend(unique_month_days)
if (12, 31) not in all_boundaries:
		all_boundaries.append((12, 31)) # end of year
all_boundaries = sorted(set(all_boundaries))

# create discrete daytypes [start, end)
for i in range(len(all_boundaries)-1):
		start_month, start_day = all_boundaries[i]
		end_month, end_day = all_boundaries[i + 1]
		
		daytype.add(DayType(
			month_start=start_month,
			day_start=start_day,
			month_end=end_month,
			day_end=end_day
		))
			
if min_delta_days >= 1:
	dailytimebrackets.add(
		DailyTimeBracket(
			hour_start=dt_time(0, 0, 0),
			hour_end=dt_time(23, 59, 59, 999999),
			name="ALL-DAY"
		)
	)
	# timeslices = daytype x dailytimebracket
	# = {D, "ALL-DAY"} for D in daytype
```
4. **Case 4:** sub-daily resolution
```python
# if subdaily
# k daytypes per year, m dailytimebrackets per day = k*m timeslices per year

# add day boundaries to discrete times
unique_times = sorted(set(ts.time() for ts in snapshots))
if dt_time(0, 0, 0) not in unique_times:
	unique_times = [dt_time(0, 0, 0)] + unique_times

# create discrete dailytimebrackets [start, end)
for i in range(len(unique_times)):
	start = unique_times[i]
	if i + 1 < len(unique_times):
		end = unique_times[i + 1]
	else:
		# Last bracket extends to end of day
		end = dt_time(23, 59, 59, 999999)
	
	dailytimebrackets.add(
		DailyTimeBracket(
			hour_start=start,
			hour_end=end
		)
	)
# timeslices = daytype x dailytimebracket
# = {D, H} for D in daytype for H in dailytimebracket
```
5. generate timeslices from `daytype` x `dailytimebracket` (note: `season` is empty)
6. associate data from `snapshots` to the relevant `timeslice`
7. fill empty `timeslices` with either default from config or forward-fill or some statistic for same `dailytimebracket` / `daytype` / `year`

## Example
- irregular spacing
- sub-daily resolution
- multi-year
```python
snapshots = pd.DateTimeIndex(
	[2020-01-02 08:00:00,
	 2020-01-02 18:00:00,
	 2020-01-03 06:00:00,
	 2020-01-03 14:00:00,
	 2020-01-03 22:00:00,
	 2020-10-01 12:00:00,
	 2021-06-01 09:00:00,
	 2025-04-01 18:00:00]
)
```

- find all unique `years`
```python
years = [2020, 2021, 2025] # all unique years in snapshots
```
- construct `dailytimebrackets` based on highest resolution of unique times
	- note that these are half-open intervals
	- we can calculate the number of hours represented by each `dailytimebracket`
```python
dailytimebrackets = [ # constructed from highest resolution across all days
	00:00:00 - 06:00:00, # from start of day (= 6 hours)
	06:00:00 - 08:00:00, # (= 2 hours)
	08:00:00 - 09:00:00, # (= 1 hour)
	09:00:00 - 12:00:00, # (= 3 hours)
	12:00:00 - 14:00:00, # (= 2 hours)
	14:00:00 - 18:00:00, # (= 4 hours)
	18:00:00 - 22:00:00, # (= 4 hours)
	22:00:00 - ENDOFDAY # to end of day (= 2 hours)
] # (total = 24 hours)
```
- construct `daytypes` based on highest resolution of unique dates
	- each date that appears in `snapshots` gets its own `daytype`
	- each date not in `snapshots` is part of a `daytype` group
	- note that these are closed intervals and they exclusively do not overlap
	- we can calculate the average number of days represented by each `daytype`
```python
daytypes = [ # constructed from highest resolution across all years
	01-01 - 01-01, # from start of year (= 1 day)
	01-02 - 01-02, # (= 1 day)
	01-03 - 01-03, # (= 1 day)
	01-04 - 03-31, # (= 28+28.25+31 days)
	04-01 - 04-01, # (= 1 day)
	04-02 - 05-31, # (= 29+31 days)
	06-01 - 06-01, # (= 1 day)
	06-02 - 09-30, # (= 29+31+31+30 days)
	10-01 - 10-01, # (= 1 day)
	10-02 - 12-31 # to end of year (= 30+30+31 days)
] # (total = 365.25 days)
```
- `timeslice` = `daytype` x `dailytimebracket` for each `year`
	- we can calculate the average number of hours represented by each `timeslice`
	- `len(timeslices) = len(daytypes) * len(dailytimebrackets)`
	- note that each `timeslice` will inherit the half-open interval from its `dailytimebracket`
	- this construction ensures that `timeslices` are AT LEAST as high in resolution as `snapshots`
```python
timeslices = [
	# daytypes[0]
	01-01 - 01-01 && 00:00:00 - 06:00:00, # (= 1*6 hours)
	01-01 - 01-01 && 06:00:00 - 08:00:00, # (= 1*2 hours)
	..., # repeat for each dailytimebracket
	
	# daytypes[1]
	01-02 - 01-02 && 00:00:00 - 06:00:00, # (= 1*6 hours)
	..., # repeat for each dailytimebracket
	
	# daytypes[2]
	01-03 - 01-03 && 00:00:00 - 06:00:00, # (= 1*6 hours)
	..., # repeat for each dailytimebracket
	
	# daytypes[3]
	01-04 - 03-31 && 00:00:00 - 06:00:00, # (= 87.25*6 hours)
	01-04 - 03-31 && 06:00:00 - 08:00:00, # (= 87.25*2 hours)
	..., # repeat for each dailytimebracket
	
	..., # repeat for each daytype
	]
```
- we assume `snapshots` represent data from [START, END)
	- as `timeslices` cover the whole year, we need to add start-of-year and end-of-year bounds for all unique years in `snapshots`
```python
bounds = [
	2020-01-01 00:00:00, # add start-of-year
	2020-01-02 08:00:00,
	2020-01-02 18:00:00,
	2020-01-03 06:00:00,
	2020-01-03 14:00:00,
	2020-01-03 22:00:00,
	2020-10-01 12:00:00,
	2020-12-31 ENDOFDAY, # add end-of-year
	2021-01-01 00:00:00, # add start-of-year
	2021-06-01 09:00:00,
	2021-12-31 ENDOFDAY, # add end-of-year
	# skip 2022, 2023, 2024 as there are no snapshots for these years
	2025-01-01 00:00:00, # add start-of-year
	2025-04-01 18:00:00,
	2025-12-31 ENDOFDAY # add end-of-year
]
```
- based on this, we can figure out the data filling pattern
	- data in `snapshots[0]` fills all times in [bounds[1], bounds[2])
	- data in snapshots[1] fills all times in [bounds[2], bounds[3])
	- ...
	- data in snapshots[4] fills all times in [bounds[5], bounds[6])
	- data in snapshots[5] fills all times in [bounds[6], bounds[7]) AND [bounds[7], bounds[8]) AND [bounds[8], bounds[9])
	- data in snapshots[6] fills all times in [bounds[9], bounds[10]) AND [bounds[10], bounds[11]) AND [bounds[11], bounds[12])
	- data in snapshots[7] fills all times in [bounds[12], bounds[13])
	
	note the half-open interval
	we always assume that the last snapshot runs till the end of the final year
	
- based on bounds, we know how many hours each snapshot represents
	- `snapshots[0]` fills 10 hours
	- `snapshots[1]` fills 12 hours
	- `snapshots[2]` fills 8 hours
	- `snapshots[3]` fills 8 hours
	- `snapshots[4]` fills 6518 hours
	- `snapshots[5]` fills 5829 hours
	- `snapshots[6]` fills 7305 hours
	- `snapshots[7]` fills 6582 hours
	
	total hours represented: 26272

- now we need an algorithm that matches the `snapshots` to the `timeslices`
```python
# create endpoints array by adding end of last year to snapshots
endpoints = snapshots.append(datetime(snapshots[-1].year, 12, 31, ENDOFDAY))

for i, e in enumerate(endpoints[:-1]):
	# get lower and upper bounds
	# note: il and iu may not be consecutive
	il, iu = bounds.find(e), bounds.find(endpoints[i+1])
	l, u = bounds[il], bounds[iu]
	
	idxs = [] # store indices of relevant timeslices
	for j, ts in enumerate(timeslices):
		# find all timeslices after lower and before upper
		if datetime(l.year, ts[0]) >= l and datetime(u.year, ts[1]) <= u:
			idxs.append(j)
	
	# check hours represented match with tolerance
	# sum of timedeltas ensures we only count represented years
	s_hours = sum(timedelta(bounds[il:iu][1:] - bounds[il:iu][:-1]))
	ts_hours = sum([len(ts) for ts in timeslices[idxs]])
	# timeslices are averaged over leap years
	# in the worst case, we are off by 0.75 days * 24 hours * years in bounds
	if abs(s_hours - ts_hours) > 0.75*24*(s_hours%365 + 1):
		raise ValueError(f"Representation Mismatch! snapshot hours = {s_hours}, timeslice hours = {ts_hours}, timeslices = \n{timeslices[idxs]}") 
```
- therefore mappings have the following structures:
	- note that each `snapshot` maps to at least one `timeslice`
	- note that each `timeslice` maps from at least one `snapshot`
```python
snapshot_to_timeslice_index: Dict[int, List[int]]
timeslice_to_snapshot_index: Dict[int, int]
```

**Workflow 2: `timeslice` >> `snapshots`**
`timeslice` = `season` x `daytype` x `dailytimebracket`
```python
for s in sorted(seasons):
	for d in sorted(daytypes):
		for t in sorted(dailytimebrackets):
			snapshots.append(timeslice[s][d][t])
```


## Simulation
1. Uniform, high resolution timeseries >> discrete, non-uniform `snapshots` (PyPSA)
2. Uniform, high resolution timeseries >> discrete, non-uniform `timeslices` (OSeMOSYS)

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

**Workflow 2: Factorization**
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
