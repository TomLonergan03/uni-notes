Hardware architectural support can simplify OS tasks, e.g. virtual memory requires a memory management unit - a hardware translator between virtual and physical memory addresses.

One hardware supported system is distinguishing between privileged and unprivileged modes, where user processes run without the ability to perform certain actions, e.g. directly accessing a device, and must instead use a **syscall** to request the OS to perform an action with that device.
# Privileged instructions
Certain instructions can only by called by the OS, these are called privileged instructions. This means that only the OS can:
- Directly access some classes of IO devices
- Manipulate memory state management
	- Page table pointers, TLB loads, etc.
- Manipulate special "mode bits", such as interrupt priority level
These provide safety and security, as they can easily be used maliciously.

The CPU knows that a privileged instruction can be run only when a status bit is set in a protected processor register. An unprivileged instruction can only set this bit by performing a syscall, which calls OS code which responds to the request and sets the bit back to 0 before returning to the user code.
If code running in user mode attempts to execute a privileged instruction then the "Privileged Instruction" exception is triggered.