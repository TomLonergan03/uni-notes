**Memory** is volatile RAM connected directly to the CPU. It is a large byte addressable array, starting with address $N$ ($0\text{x}0000$ in x86) and (generally) ending with address $N+\text{TOTAL\_RAM}$. It contains all the code that is running on the CPU, along with data for that program.
# Goals of memory management
An operating system manages the allocation of memory resources among processes and the OS. The OS usually wants to simplify memory utilisation by providing convenient abstractions for processes and programmers, and to isolate processes from the OS and each other.
# Basics
Main memory and CPU registers are the only storage devices that the CPU can access directly. Persistent storage must be loaded into memory before the CPU can interact with it. A register access can be done in one CPU cycle, while a memory access takes many, causing the CPU to stall. Caches sit between main memory and CPU registers, and can be accessed faster reducing CPU cycles to access commonly used memory. Cache is invisible to the assembly programmer, and is handled by the hardware.
# No memory abstraction
If we only run one program, then the program can directly interact with RAM, e.g. in some embedded systems. If there is also an OS then the program can change values in the operating system, which is likely to be unstable and insecure. If we have multiple programs with no memory abstraction then each program can also interfere with other programs' data or code, which is similarly insecure and risky.
It is also difficult for a program to find its data, as if programs are started in different orders then hardcoded pointers will not work as the programs will likely not be loaded into the same locations on multiple executions.
## Binding
One way to solve the pointer problem is **binding**, the process of mapping variables to memory addresses. Binding occurs in stages:
1. The compiler binds symbolic addresses to relocatable addresses (base + offset)
2. The linker can bind relocatable addresses to absolute addresses if the final memory location is already known
3. The loader binds relocatable addresses to absolute addresses after the final memory location is decided
This is expensive on loading, and doesn't prevent a program from interfering with other processes.
## Base and limit registers
To prevent programs from accessing memory they aren't supposed to, each program could have an OS set base and limit register. These registers would define the range of memory a process can access, and if it tries to access an address outside that region it traps to the OS. This still requires address relocation though.
# Memory abstraction
An **address space** is an abstraction of the physical address space, which allows multiple processes to access what they perceive as the same address, but that doesn't correspond to the same physical address. Processes therefore use **logical addresses**, which are translated by OS configured hardware into physical addresses.
A program therefore has a contiguous logical address space that starts at 0, which maps to a contiguous physical address space that starts at any point in RAM.
## Memory management unit (MMU)
The **memory management unit (MMU)** is the physical hardware which translates logical addresses to physical addresses.