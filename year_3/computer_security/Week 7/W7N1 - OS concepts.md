> [!note]
> This section of the course focuses on operating system security specifically in the context of the x86 architecture and Unix based operating systems

# Purpose of operating systems
An OS is the interface between the users/applications of a computer and that computer's hardware resources, such as storage, memory, CPU time, IO devices, and network interfaces.
- **Multiple users:** an OS should be able to allow for multiple users with different levels of access, and be able to isolate them from each other.
- **Multitasking:** an OS must be able to run many applications simultaneously, and to isolate them from each other.

# Basic Unix architecture
![[w7n1UnixArchitecture.png]]
The Unix **kernel** is the core of the operating system, which supports secure sharing of low level resources, by limiting how applications can access them.

There are two **modes of execution:**
- **User mode:** where access to resources is only possible through syscalls to the kernel
- **Kernel mode:** where resources are directly accessed

**System calls** (or **syscalls**) are usually contained in a collection of programs, such as a library like [libc](https://en.wikipedia.org/wiki/C_standard_library). Syscalls include `open()`, `close()`, `fork()`, `exec()`, etc.

# Processes
A process is an instance of a program which is currently executing. When a program is executed, it is loaded into RAM, and given a unique **process ID (PID)**. Each PID is then associated with its CPU time, memory usage, **user ID (UID)**, program name, etc.
A process may control other processes, via forking, and child processes inherit context from their parents. ^260d9e
# x86 process memory layout
![[w7n1ProcessMemoryLayout.png]]
The stack contains the variables of a process, the heap contains dynamically allocated memory, static data contains objects which last for the duration of the process such as global variables and static class members, and text contains the actual instructions that the process will execute.

# Virtual memory
![[w7n1VirtualMemory.png]]
When there is not enough RAM on a system, also in order to isolate processes memory from each other, **virtual memory** is used. Virtual memory works by mapping logical addresses to physical addresses, and in the case of not enough RAM being present in a system, some of those physical addresses can be stored on a storage drive.