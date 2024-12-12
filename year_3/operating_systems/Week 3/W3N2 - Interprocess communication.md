For complex work, OSes generally require a method to allow multiple [[W3N1 - Processes|processes]] to share information between them. This allows for modular components as separate processes, and enables the use of parallel hardware for computational speedups.

There are 2 main approaches:
# Shared memory
![[w3n2sharedMemory.png]]

Processes can choose to share part of their address space (on POSIX compliant systems by using the `shm_open()` syscall), disabling the OS' protection of that region. This allows for fast access - as it's not mediated by the OS - but requires the application programmer to fully manage data transfer and decide on the format.
# Message passing
![[w3n2messagePassing.png]]
Message passing uses one of a number of OS provided communication channels (which introduces overhead as the OS mediates the communication, but simplifies work for the application programmer and allows for sharing information between different networked machines). A message passing facility provides at least 2 operations: `send(message)` and `receive(message)`.
## Naming
Communicating processes must be able to refer to each other. This is either:
1. **Direct**
	- **Symmetric**: there's an explicit name for who to send to and who to receive from
	- **Asymmetric**: there is either an explicit process to send to or receive from, e.g. sending a message to everyone but each process has to choose to receive from the sender, or sending a message to a specific process but the receiver receives from all processes, with a sender ID returned to the receiver
2. **Indirect**: no need to know what process is the sender/receiver, this can be using mailboxes, such as POSIX mailbox, where a process can send a message to a specific mailbox or receive it from a specific mailbox
## Synchronisation
Passing a message can be either **blocking/synchronous** or **nonblocking/asynchronous**, and sometimes combinations may be offered, e.g. blocking send, nonblocking send, blocking receive, nonblocking receive.
**Rendezvous** protocols have both `send()` and `receive()` strictly blocking, which avoids the need to buffer sent messages or handle messages that are never received.
## Buffering
Messages sent can reside in temporary buffers/queues of varying lengths:
- **Zero capacity**: messages are not buffered, instead the sender must block until the recipient receives the message
- **Bounded capacity**: there is a finite length $n$, in which at most $n$ messages can be stored. If the queue is not full when a message is sent, it is added to the queue and the sender continues execution, if it is full, then the sender has to wait.
- **Unbounded capacity**: there is an infinite queue (limited only by hardware), so the sender never blocks
## Pipes
A pipe is a conduit that allows a parent and child process to communicate using a pair of file descriptors. A write to one FD outputs on the other process' corresponding FD.
## Sockets
Sockets are endpoints for communication, and are identified by an IP address concatenated with a port number. Sockets can be used for communicating between networked machines.
(Also see [[W3N1 - Network applications#Sockets|here]])
## Signals
Signals are an OS mechanism used to notify a process (so are one way only). These can come either from the OS, e.g. division by 0 errors or timers, or from other processes, e.g. process management like `kill` or synchronisation such as POSIX RT signals