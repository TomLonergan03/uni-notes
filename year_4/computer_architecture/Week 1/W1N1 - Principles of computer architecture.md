# What is computer architecture
Computer architecture is the design of an **instruction set architecture (ISA)**, the **microarchitecture** that implements that ISA, and the hardware implementation of that microarchitecture.
# Metrics of interest
- Performance
- Cost
- Reliability
- Power
- Area (how many mm$^2$ a chip covers)
## Performance
### Measuring performance
- **Execution/response time**: the overall time for a given computation e.g. a full program execution, or one transaction
- **Latency**: time taken to complete a given task e.g. memory latency, IO latency, instruction latency
- **Bandwidth/throughput**: rate of completion of tasks e.g. memory bandwidth, transactions per second, **million instructions per second (MIPS)**, **floating point operations per second (FLOPS)**
	- MIPS/FLOPS must be used with caution, as e.g. one architecture may use a few instructions that each do a lot, while another uses many small instructions. Each architecture would have different MIPS/FLOPS scores, but may have equivalent execution times.

**Benchmarks** are used to measure the performance of an architecture. Forms of benchmarks include:
- Toy benchmarks: tiny programs to measure a specific part of a processor
- Synthetic benchmarks: artificial tasks that mimic some aspect of real programs, e.g. making a certain ratio of memory accesses to arithmetic operations
- Kernels: very small loops that test a specific action, e.g. matrix multiplication
- Real programs: can be more realistic, as it is an actual task that a processor may run. Can be hard to port to new architectures, and choice of program can result in large relative changes between different processors
- Benchmarking suites: combine a number of different benchmarks into one tool, and often produces one overall score for a run. Examples include SPEC INT/FP, EEMBC Coremark, CloudSuit

$A$ is $n$ times faster than $B$ means $\frac{\text{Execution time of }B}{\text{Execution time of }A}=n$.
$A$ is $m\%$ faster than $B$ means $\frac{(\text{Execution time of }B-\text{Execution time of }A)\times100}{\text{Execution time of }A}=m\%$.

### Improving performance
- **Parallelism** on all levels:
	- Increase number of processors in a system
	- Run more instructions simultaneously on one processor (e.g. pipelining)
	- Operate on more bits simultaneously in one circuit(e.g. carry-lookahead ALU)
- **Locality**:
	- **Spatial locality**: if we access a memory address $A$, then it is highly likely we will access other memory addresses close to $A$, e.g. operating on an array, or as a program generally executes instruction by instruction with infrequent jumps ^00d1cb
	- **Temporal locality**: if we access a memory address at time $T$, it is likely we will access it again near to $T$
	- Both of these are used in **caches**, which keeps recently used data close to the processor
- Focus on the common case:
	- Generally, we have a few common tasks, and many rare tasks. This means that we can focus on making the common cases fast, while everything else just needs to be correct
### Factors that affect CPU performance
- **Instruction count (IC)**: the number of instructions needing executed for a task, determined by ISA and the compiler used
- **Cycles per instruction (CPI)**: the number of cycles taken to execute an instruction, determined by ISA and microarchitecture
- **Clock time**: the time per clock cycle, determined by microarchitecture and technological limits
$$
\text{CPU time}=\sum^{n}_{i=1}(CPI_i\times\frac{IC_i}{IC})\times\text{clock time}
$$
where
$IC_i=IC\text{ for instructions in instruction group }i$
$CPI_i=CPI\text{ for instructions in instruction group }i$
### Improving performance
- IC can be improved with:
	- Compiler optimisations, e.g. folding constants together at compile time and propagating them through the program
	- Using more complex instructions in the ISA
- CPI can be improved with:
	- Microarchitecture improvements, e.g. pipelining, out-of-order execution, branch prediction
	- Compiler optimisations, e.g. instruction scheduling
	- Using simpler instructions in the ISA
- Clock period can be improved with:
	- Smaller transistors
	- Simple, easy to decode instructions
	- Simpler microarchitectures