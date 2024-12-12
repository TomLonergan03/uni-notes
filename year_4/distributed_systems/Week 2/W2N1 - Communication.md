Processes/threads/VMs/nodes perform **computation**, but they must also **communicate** information to form a distributed system.

# Categories of communication paradigms
- Same address space
	- Within a process
	- E.g. global variables, procedure calls
- Different address spaces, same computer
	- E.g. files, shared memory, signals
- Different address spaces, multiple computers
	- E.g. distributed shared memory, message passing, RPC, sockets

## Message passing
**Message passing** consists of participants actively sending or receiving data between themselves, with no shared memory space between the nodes.
### Sockets
**Sockets** are software structures for sending and receiving data across a network.
![[w2n1sockets.png]]
## Remote procedure calls (RPCs)
A **remote procedure call (RPC)** allows remote services to be called as procedures. Parameters are passed by marshalling/packing them into a message to be transmitted.
An **interface definition language (IDL)** is used to define how data is represented.
The **binder** has a list of servers and their exported interfaces, and when a client calls a procedure the binder decides which server will be called for that procedure