2025-12-08

GitHub repo: https://github.com/ImperialCollegeLondon/RCDS-profiling-and-optimisation-in-python

reduce the amount of time that it takes you as a developer to write the code

profiling code
- too short = subject to noise
- too long = difficult to validate the effect of code optimization
- 10 s - 1 min is appropriate

how can I run this piece of code in a way that is representative of how my entire code will run without waiting too long for it

profiling tools
- runtime profiling = how long it takes the code to run
	- timing
	- cProfile
	- line_profiler
		- runs alongside code run to make measurements
		- will potentially slow down code significantly
		- can -f multiple functions to profile them all
```python
cProfile.run("COMMAND_TO_EXECUTE", sort="tottime")
```
```python
%load_ext line_profiler
%lprun -f FUNCTION_TO_PROFILE COMMAND_TO_EXECUTE
```
- memory profiling = usage of memory as data is created and discarded
	- pympler
```python
from pympler import summary, muppy
mem1 = muppy.get_objects()
##### code here #####
mem2 = muppy.get_objects()
summary.print(
	summary.get_diff(summary.summarize(mem1), 
					 summary.summarize(mem2)
	)
)
```

optimization
- algorithm choice
	- use big O notation to determine best choice for runtime, insertion/deletion, memory
	- reduce the number of calculations using algebraic tricks
		- find the least expensive combination of math operators which return the desired value
		- arithmetic operators are cheaper than complex ones
		- use specialized versions of functions where possible
- caching
	- cache variables by pre-calculating and storing (perhaps in a list)
	- cache functions called with same values using `lru_cache` decorator from `functools`
		- useful when small number of values is passed as argument but the number of function calls is high
		- has overhead so if there is a way to save as a list and look up value later, do it
```python
from functools import lru_cache

@lru_cache(maxsize=n)
def func(x):
	# function code here
	return y
```
- optimize loops
	- optimize inner loop first as it runs more times than any other
	- eliminate a loop if possible
	- list comprehension, e.g. `[i**2 for i in range(10000)]`
	- `map()` function to apply function to list or range, e.g. `map(math.sqrt, range(n))`
	- use `break` to exit early where possible
- parallelism
	- degree of speed-up depends on the number of cores available
```python
from joblib import Parallel, delayed
import multiprocessing

n_core = multiprocessing.cpu_count()
tallies = Parallel(n_jobs=n_core)(delayed(func)(n//n_core) for i in range(n_core))
```