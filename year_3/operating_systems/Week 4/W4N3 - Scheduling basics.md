A computer is often running hundreds (Linux) or thousands (Windows) of [[W4N1 - Threads|threads]]. The scheduler decides which thread gets to run next. Simply, it "just picks one" from the [[W3N1 - Processes#State queues|ready queue]], but it is much more complex and smarter than that. The active thread could change any time we switch to the OS - any interrupt, syscall, or exception.
![[w4n3schedulerVsDispatcher.png]]
The scheduler uses a policy (an algorithm) to determine which thread runs next. The actual task switching is performed by the **dispatcher**, which saves the outgoing process' state into that process' [[W3N1 - Processes#Process representation in the OS|PCB]] then restores the incoming process from its PCB.
# Process/thread behaviour
> [!abstract] Note
> I'm going to use thread to mean process/thread in this section, as they both exhibit similar behaviour

Thread execution consists of cycles of CPU execution (**CPU bursts**) and IO waits (**IO bursts**).
![[w4n3bursts.png]]
While a thread is waiting for IO, another thread can be run. 
There are two(ish) types of threads:
- **CPU bound** threads: where most of the time is spent in long CPU bursts
- **IO bound** threads: where most of the time is spent waiting for IO
![[w4n3typesOfThreads.png]]
# Scheduling goals
There are many (conflicting) performance goals:
- Maximise CPU utilisation
- Maximise **throughput** (processes completed per second)
- Minimise **turnaround time** (time from submission of task to completion)
- Minimise **waiting time** (total period spent waiting in the ready queue)
- Minimise **response time** (time from submission of task to response being produced)
- Minimise **energy** (joules per instruction) subject to some constraint (e.g. hit a minimum frame rate)
In most cases we optimise for average performance, but sometimes it is also desirable to minimise the worst case.
## Fairness
Generally, we want to be "fair" - but there is no single definition of fair. We could aim for equal CPU consumption over some timeframe, or fair per user/process/thread, but any of these have obvious cases where that is not desired. Also, sometimes the goal is to be unfair, e.g. having a priority system where certain requests are favoured.
# Classes of schedulers
- **Batch** schedulers
	- Aims to maximise throughput/utilisation
	- Used for e.g. rendering, MapReduce jobs, auditing large amounts of data
- **Interactive** schedulers
	- Aims to minimise response time
	- Used for e.g. an operating system based around a graphical interface
- **Real time** schedulers
	- Must meet certain deadlines, e.g. a task must run at exactly 10 kHz
	- Used in embedded systems
# Modelling scheduling problems
![[w4n3modelScheduling.png]]
Our model will consist of schedulable units (generally processes or threads, but could be many things such as disk ops) and resources which can be allocated to those units (generally CPU time, or could be disk time/memory space etc.)
# Preemptive vs non-preemptive scheduling
- **Non-preemptive** scheduling switches tasks only when one completes, voluntarily yields, or makes a blocking call
- **Preemptive** scheduling may pause a task mid-execution. This is non-voluntary, and happens due to a clock generating interrupts at a certain frequency
