# Hardware
| Year | System | Description |
| ---- | ---- | ---- |
| 1951 | Wolverhampton Instrument for teaching computing from Harwell (WITCH) | The first digital computer, has no operating system and instead runs a single calculation |
| 1960s | IBM starts producing mainframe computers | Large central computers |
| 1970s | Minicomputers are released |  |
| 1980s | Microprocessors and desktop workstations are created, along with local area networking and the Internet |  |
| 1990s | PCs spread, along with modern tech companies, e.g. Microsoft, Intel, Dell |  |
| 2000s | Internet services and cloud computing arise, mobile computing, computers become ubiquitous |  |
| 2010s | Data intensive computing, sensor networks, IoT |  |
# Operating systems
Early computers had a structure similar to this:
![[w1n2earlyComputerStructure.png]]They consisted of an input device, a printer, memory and sometimes a permanent storage disk.

## OS as a linked library
Early "operating systems" were libraries that a program would be linked to. A program would be loaded into memory, and executed, with the "OS" providing functions for controlling the disk, printer, etc.

At this stage a computer could still only run one program at a time, and only one user could use it at a time.
## Asynchronous IO
The disk was very slow (it could take several seconds to perform a simple read or write operation). This led to the creation of [[W1N1 - Operating systems basics#Hardware devices|disk controllers]], which meant that programs could now start an IO request, do some other computation while the op happens, then use the result once it is ready.
## Multiprogramming
Multiprogramming OSes could run multiple jobs simultaneously, and overlaps the IO of each job - while one job waits for IO completion, another can use the CPU. This also allows for getting rid of asynchronous IO within individual jobs, which makes application programming easier.
## Timesharing
A timesharing OS allows multiple terminals to be running on the same machine, with each user having the illusion of having the entire machine to themselves. This is done using **timeslicing**, by dividing CPU time equally between users, and switching between them fast enough that no user notices the slowdown.

One of the first timesharing systems was the MIT CTSS system in 1961, which had only one user memory-resident at a time and only 32KB of memory. This was followed by the MIT Multics system in 1968, which was the first large timeshared system, and nearly all OS concepts can be traced back to it!
## Parallel systems
Some applications can be written as multiple parallel processes/threads, which can speed up execution by running the threads simultaneously on multiple CPUs. This requires OS and language primitives for dividing the program up, as well as for fast communication between threads.
## Personal computing
Up until the 70s, computers were aimed at organisations, but with the arrival of personal computing more intuitive input devices and applications were desired. This lead to the creation of the mouse, along with many systems such as graphical interfaces and bit mapped displays. This meant that now the display is also an OS managed resource.
## Progression of programming abstractions
![[w1n2programmingAbstractions.png]]

# When a single computer is not enough
Eventually, workloads exceeded what can be run or stored on a single machine. There are many attempts to solve this problem
## Distributed OSes
A distributed OS consists of one operating system which runs over many network connected computers, and handles interprocess communication, load balancing, access control, etc.
![[w1n2distributedOs.png]]
These didn't catch on, as they are very complex and difficult to debug.
## Client/server computing
In this model, a client opens a connection to a server and makes requests to it, which the server responds to. Here, the client and server each have their own independent OS, which controls communication and access control between the two systems.
## Peer-to-peer (p2p) systems
In a p2p system, each computer acts as both a client and a server, and connects to many other systems in the network, with no "master system" controlling all the others. These systems are commonly used for illegal file sharing such as piracy, as there is no central figure to be charged.
## Cloud computing
Cloud computing works by companies who own many machines and rent access to small numbers of those machines to many different organisations.