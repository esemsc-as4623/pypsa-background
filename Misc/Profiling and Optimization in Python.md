2025-12-08

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