# Communicating with devices
  A device has a device controller which is responsible for e.g. sending the commands to physically read a hard drive sector. The CPU communicates with these controllers by reading and writing to their registers and buffers.
  There are 2 main ways to access those buffers:
  - **IO ports:** each controller registers an IO port number, and there are special x86 instructions to access IO ports, e.g to write to a device the instruction `OUT PORT, REG` writes the contents of a register to a port
	  - IO instructions are [[W1N3 - Syscalls#Privileged instructions|privileged]], so can only be called by the kernel.
	  - IO ports have a separate address space to memory
	    ![[w2n2ioPorts.png]]
  - **Memory mapped IO:** instead of having separate IO and memory spaces, we can map buffers into the address space. Each buffer/register has a unique address, which has *no physical RAM for that address*. Such addresses are often at the top of the physical address space.
    ![[w2n2memoryMapping.png]]
# Direct memory access (DMA)
The CPU can request data from an IO controller one byte at a time. This is fine for small data transfers, but wastes a lot of time for large transfers. This is fixed using a **DMA controller** which can transfer data for the CPU, either between memory and an IO device, or between 2 IO devices.
# Device drivers
Every device model will have its own semi-unique set of buffers and registers. It is impractical for an OS to ship with drivers for every device in existence, so instead manufacturers write drivers for their devices, with these drivers conforming to a standard interface for its type.
# IO services provided by the OS
The OS provides these IO services (among other things):
- management of the namespace for files and devices
- access control to files and devices
- operation control (e.g. a modem cannot `seek()`)
- file-system space allocation
- device allocation
- buffering, caching, and spooling
- IO scheduling
- device monitoring, error handling, and failure recovery
- driver configuration and initialisation
- device power management
# Life cycle of an IO request
![[w2n2ioLifecycle.png]]