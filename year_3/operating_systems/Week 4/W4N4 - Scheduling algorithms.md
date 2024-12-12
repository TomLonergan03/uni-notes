There are many possible [[W4N3 - Scheduling basics|scheduling]] algorithms. We will look at some here
# First-come first-served (FCFS)
Here, processes are simply assigned to the CPU in the order they arrive. It is non-preemptive. A real-world example is "scheduling" of people in a line for something.
## FCFS example
If we have 3 processes:

| Process | CPU time |
| :--: | :--: |
| P1 | 24 |
| P2 | 3 |
| P3 | 3 |
If they arrive at the queue in the order $P1, P2, P3$, then the turnaround times are $P1 = 24, P2 = 27, P3 = 30$, with an average turnaround time of $(24+27+30)/3=27$
If instead they arrive in the order $P2, P3, P1$, then the turnaround time P1 $= 30, P2 = 3, P3 = 6$, for an average turnaround time of $(3+6+30)/3=13$.

This is an example of the **convoy effect**, where small processes are significantly delayed by waiting for larger processes to finish.
## FCFS drawbacks
- Average response time can be poor due to convoy effect
- May lead to poor utilisation of resources
	- E.g. a CPU intensive job prevents an IO intensive job from carrying out a small bit of computation, which means that the IO subsystem is being underutilised
# Shortest job first (SJF)
Here, we estimate the execution time required for each process. The process with the shortest amount of CPU time is scheduled first. This has two variations, one [[W4N3 - Scheduling basics#Preemptive vs non-preemptive scheduling|non-preemptive]], and one [[W4N3 - Scheduling basics#Preemptive vs non-preemptive scheduling|preemptive]] where if a new process arrives with a CPU time of less than the current executing process has remaining, it is preempted (this is **shortest remaining time next (SRTN)**).
## SJF example
| Process | Arrival time | CPU time |
| :--- | ---- | ---- |
| $P1$ | 0 | 7 |
| $P2$ | 2 | 4 |
| $P3$ | 4 | 1 |
| $P4$ | 5 | 4 |
$P1$ arrives first and uses the first 7 units of CPU time, then $P3$ is scheduled next as it is the shortest job, then $P2$, then $P4$. Turnaround times are $P1=7,P2=12-2=10,P3=8-4=4,P4=16-5=11$ for an average turnaround time of $(7+10+4+11)/4=8$ and makes 3 context switches
## SRTN example
We execute $P1$ for 2 units, then $P2$ for 2 units, then $P3$ for 1 unit, then $P2$ for 2 units, then $P4$ for 4 units, then finally $P1$ for 5 units for an average turnaround time of $(16+5+1+6)/4$ and 5 context switches
## SJF drawbacks
True SRTN is optimal, but it can only be approximated, as its not always possible to determine the duration of a CPU burst
# Round robin (RR)
Each process is allowed to run for a specific time interval (a **quantum**). After this time has elapsed the process is preempted, added to the end of the ready queue, and the next process is scheduled. If the process terminates or blocks before this time it is added to a wait queue and the next process is scheduled.
## RR example
| Process | CPU time |
| ---- | ---- |
| $P1$ | 53 |
| $P2$ | 8 |
| $P3$ | 68 |
| $P4$ | 24 |
![[w4n4roundRobinExample.png]]
If we set the quantum to be 20, then the waiting times are: $P1=(68-20)+(112-88)=72$, $P2=(20-0)=20$, $P3=(28-0)+(88-48)+(125-108)=85$, $P4=(48-0)+(108-68)=88$
For an average waiting time of $(72+20+85+88)/4=66.25$ and an average turnaround time of $(125+28+53+112)/4=104.5$
## Choosing a time quantum
Context switching may impact the choice of time quantum, e.g. if a context switch takes 1ms and the time quantum is 4ms then 20% of the time is lost to context switching. Typically, a context switch is in the order of microseconds, while the quantum is 1KHz (every 1ms)
## Advantages and disadvantages
- Advantages
	- A solution to fairness and avoids starvation
	- Equally spreads CPU time across jobs
	- Low average waiting time when job lengths vary
	- Good for [[W4N3 - Scheduling basics#Classes of schedulers|interactivity]] if there's a small number of jobs
- Disadvantages
	- Context-switching time adds up for long jobs
	- Shares CPU cache between all jobs, so is slower than FCFS even with 0-cost context switches
# Priority (PRIO)
Each process is given a priority, and the highest priority process is run on a FCFS basis.
![[w4n4priority.png]]
Priority can be assigned in 2 ways:
- **Statically**: each process is given a priority by the developer or based on which user is using it or based on how much a user paid to run the job (or many other ways)
- **Dynamically**, based on how much they run vs IO
	- e.g. $\text{Priority}=1/f$ where $f=\text{size of last quantum}$, so the longer a process runs for the lower its priority, and the process that runs for the shortest period (i.e. is [[W4N3 - Scheduling basics#Process/thread behaviour|IO bound]]) runs next
## Static PRIO Example
| Process | CPU time | Priority |
| ---- | ---- | ---- |
| $P1$ | 10 | 3 |
| $P2$ | 1 | 1 |
| $P3$ | 2 | 4 |
| $P4$ | 1 | 5 |
| $P5$ | 5 | 2 |
Turnaround times are $P1=16$, $P2=1$, $P3=18$, $P4=19$, $P5=6$ for an average turnaround time of $(16+1+18+9+6)/5=12$.
## Disadvantages
- It is possible for low priority processes to starve if there are enough higher priority processes to use all available CPU time.
- Deadlocks are possible if a low priority task has a lock on a resource that a high priority task needs
# Multiple queues (MQ)
![[w4n4multipleQueues.png]]
Similar to PRIO except each priority is run in a RR style.
## Example
| Process | CPU time | Priority |
| ---- | ---- | ---- |
| $P1$ | 10 | 3 |
| $P2$ | 1 | 1 |
| $P3$ | 2 | 4 |
| $P4$ | 1 | 5 |
| $P5$ | 5 | 2 |
If we use quantum = 2 then these are scheduled as:
![[w4n4mqScheduled.png]]
Turnaround times are $P1=19$, $P2=1$, $P3=10$, $P4=11$, $P5=6$ for an average turnaround time of $(19+1+10+11+6)/5=9.4$.
# Multilevel feedback queue (MLFQ)
This is similar to MQ, but each queue has a different time quanta, with lowest priority having a large quanta and highest priority having a small quanta. Processes start at the highest priority, and if it exceeds its quanta it is moved to a lower priority, or if it becomes interactive it moves to a higher priority. This is the scheduler used in the Linux kernel (with some tweaks).
![[w4n4multilevelFeedbackQueue.png]]
# Selecting a CPU scheduling algorithm
First you have to determine what goal the algorithm should meet, e.g. [[W4N3 - Scheduling basics#Scheduling goals|utilisation, response time, or throughput]]. Then, you evaluate options using some or all of:
- **Deterministic modelling** by evaluating performance on the known system workload
- **Queuing models** using a mathematical model
- **Simulations**
- Actual **implementation** and testing
# Multicore/multiprocessor scheduling
If there are more than 1 CPU core/CPU then there are more things to consider, including:
- **Load sharing**: where processes/threads can run in parallel and the load on each CPU should be somewhat balanced
- **Asymmetric multiprocessing**: some cores/CPUs may have different processing power/IO access, so scheduling must account for that. This is common in embedded systems
- **Symmetric multiprocessing**: all cores/CPUs can do everything
	- We can either maintain one global ready queue or each processor has its own in which case there can be problems with work sharing