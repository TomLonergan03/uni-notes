A **process** is the OS's abstraction for execution. A program is a list of instructions, initialised data, etc. while a process is a program in execution. Only one (almost, [hyperthreading](https://en.wikipedia.org/wiki/Hyper-threading) makes this more complicated) process is running on a processor core at any instant.
A process consists of:
- An address space, containing code for the running program and data (static data, heap, stack)
- A CPU state, consisting of a program counter (PC) which indicates the next instruction, a stack pointer which contains the current stack position, and other general purpose register values
- A set of OS resources, such as open files, network connections, sound channels
This is everything needed to run the program, or restart it if interrupted.
Each process is identified by a unique (almost, cgroups etc. make this more complicated), global **process ID (PID)**. ^956d06
# Process representation in the OS
The OS keeps track of all processes' states in the **process control block (PCB)**, with each process identified by its PID.
The OS keeps all of a process' execution state in (or pointed to) the PCB - PC, SP, registers, etc. - when the process isn't running, and when it is running the state is spread between the PCB and the hardware CPU registers/memory.
## The PCB in Linux
Each process contains many fields, including:
- process ID (PID)
- parent process ID
- execution state
- program counter, stack pointer, registers
- address space info– 
- UNIX user id, group id
- scheduling priority
- accounting info
- pointers for state queues
In Linux, the PCB is defined in `task_struct` (`include/linux/sched.h`), and has more than a 100 fields which occupies ~4kB.

![[w3n1linuxPcb.png]]
When a process is running, its CPU state is inside the CPU. When the OS gets controls because of a syscall/exception/interrupt the OS saves the CPU state of the running process in that process' PCB. When the OS returns the process to the running state it loads the hardware registers with values from that process' PCB. 

The act of switching the CPU from one process to another is a **context switch**. Modern systems my do 100s or 1000s of switches per second, with one switch taking (in the order of) microseconds (which is still expensive relative to thread based context switches). Choosing which process to run next is called **scheduling**.

![[w3n1processContextSwitch.png]]
## Process execution states
Every process has an **execution state**, which describes what it is currently doing:
- ready: waiting to be assigned to a CPU
- running: executing on a CPU
- waiting: waiting for an event, e.g. IO completion or a message from/completion of another process

![[w3n1processStates.png]]
(the OS has a timer running which interrupts each process every so often which allows it to schedule a new process if the first hasn't called a syscall/interrupt in that period - this is called **preempting**)
## State queues
The OS maintains a collection of queues to represent the state of all processes in the system. There is typically one queue for each state, but there may be many waiting queues, one for each type of wait (e.g. specific device, timer, message)

## PCBs and state queues
When a process is created, the OS allocates a PCB for it, initialises the PCB, and puts the PCB onto the appropriate queue. As the process computes the PCB is moved from queue to queue depending on its state. When the process is terminated the PCB may be retained for a while (to receive signals etc.), but eventually the OS deallocates it.
## Process creation
In a unix-like system, new processes are created by existing process. The created process is called the **child process**, and the creator the **parent process**. The first process (with a PID of 0) is started by the OS (this is often `init` or `systemd)`. Depending on the OS, children inherit some attributes of the parent, e.g. open file tables. On some systems, resources allocated to the parent may be divided between its children.
### UNIX process creation
UNIX processes are created using the `fork()` syscall. This creates and initialises a new PCB, initialises kernel resources with resources of the parents, and initialises registers to be the same as the parent. It also creates a new address space and initialises it with a copy of the *entire contents of the parent's address space*. The `fork()` syscall "returns twice", once into the parent where it returns the child's PID, and once into the child where it returns 0.

![[w3n1forking.png]]
#### Fork and exec
To start a new program, we first `fork` then use `exec()` to stop the current process, and load and initialise the hardware context and args for the new program, overwriting the existing process image, and places the PCB onto the ready queue.

![[w3n1forkAndExec.png]]
#### Making process creation faster
`fork` creates a copy of the parents memory space. Doing this directly is slow, as we have to allocate physical memory, set up child's page tables, copy parent's address space, and and then is likely overwritten anyway by `exec`. This can be made faster:
1. `vfork()` - an old, now uncommon, approach, which instead of semantically saying "the child's address space is a copy of the parent's" it instead says "the child's address space is the parent's". This comes with an (unenforced!) promise that the child won't modify the address space before doing an `execve()`. The parent is blocked until `execve()` is called by the child. This all saves the wasted effort of duplicating the parent's address space.
2. **Copy on write** - retains the original semantics of fork, but only copies what is necessary. It does this by creating the child's new address space, but it initialises the page tables to be the same as the parent's, meaning that no memory has yet been changed (other than the PID on the top of the stack). Both parent and child page tables are set to make all pages read-only. This means that if the parent or child writes to memory an exception occurs, and when this happens the OS copies the page, and adjusts the page tables. This means that memory is only copied when the parent and child want to update them, massively reducing the amount of memory copied.