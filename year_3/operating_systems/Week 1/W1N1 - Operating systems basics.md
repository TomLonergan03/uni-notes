# What is an operating system
An operating system is a program that acts as the intermediary between a user/program and the computer hardware, in order to manage access to resources.
It also:
- makes development of programs easier by providing higher level programming abstractions
- makes applications software more portable and versatile
- makes use of hardware more efficient by allowing sharing of hardware and software resources
- provides isolation, security, and protection between programs by limiting what whey can do
- isolates faults to single programs, and provides information on where they went wrong (e.g. segmentation fault)

# What is in an operating system
![[w1n1OsContents.png]]An OS (generally) contains a user interface, core runtime libraries (e.g. libc), and a kernel.

# The OS and hardware
An operating system mediates programs' access to hardware resources:
- computation (CPU)
- volatile storage (memory) and persistent storage (disk, etc.)
- network communications (TCP/IP stacks, Ethernet cards, etc.)
- IO devices (keyboard, display, sound card, etc.)

The OS abstracts hardware into logical resources and interfaces to those resources:
TODO

# Why bother with an OS
Application benefits:
- Programming simplicity:
	- TODO
TODO

# Hardware devices
Every hardware device has a device controller, which ^beb05c
- may move data to main memory, like the CPU
- run in parallel to the CPU
- have buffers for data (each has local memory)
Operating systems have device drivers for device controllers.

## Example - reading a file
1. An application wants to read a file on a disk
2. Application requests a file from a path from the OS
3. OS converts the path (which is an abstraction) into a physical disk address
4. OS allocates memory space to receive the data
5. OS' disk device driver sends low-level command to the disk
6. Disk reads data into internal buffer
7. Disk transfers data into memory space
8. When data transfer is finished, the disk issues an interruption
9. The OS receives the interrupt and runs the interrupt handler
10. OS notifies the application that the data is ready and in memory
