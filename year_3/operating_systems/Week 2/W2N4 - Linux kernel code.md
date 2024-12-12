# Linux code layout
In a running system, the kernel binary is typically located at `/boot/vmlinuz-X.Y.Z`.
The kernel source code is obtainable from kernel.org, the Linux GitHub mirror, etc. Some interesting folders within the repo include:
- `arch/` - architecture specific code, with one folder per supported architecture, which includes `x86`, `x86_64`, `MIPS`, `powerpc` and more
- `include/` - kernel header files, used for other programs to link against certain kernel functions
- `init/` - kernel initialisation code, although most of the boot process is architecture specific so is in `arch/`
- `mm/` - memory management
- `drivers/` - device drivers
- `ipc/` - inter-process communication
- `fs/` - filesystem
- `kernel/` - generic core kernel code
- `net/` - networking
- `block/` - block layer
- `lib/` - helper libraries
- `scripts/` - scripts used during kernel configuration or compilation
- `Documentation/` - kernel documentation text files
- `tools/` - user space programs for various purposes, e.g. debugging, tracing, performance analysis
# Exploring the source code
The easiest way to do that is using `lxr` or Linux cross reference, which can be accessed at [http://lxr.free-electrons.com/](http://lxr.free-electrons.com/) (and other places). This allows you to browse the code of different Linux versions, search for identifiers, view call chains, and quickly look up declarations and definitions.
## Cscope
`Cscope` is a command line tool for quickly navigating large C codebases.
# Programming for the kernel
An important note about the kernel is that it is entirely self contained, meaning there is no `libc`. Many of `libc`'s functions are implemented within the kernel in a cut-down form, or in ways that better suit the kernel's needs. Some examples include:
- `printk()` - similar to `printf()`, but it takes a log level value also and prints to the kernel log (printed to `/dev/kmsg`, which can be accessed from userspace using `dmesg`) instead of a shell (depending on the print's log level the message may or may not appear)
- `kmalloc` and `kfree` - similar to `malloc` and `free`, except `kmalloc` takes a `gfp` parameter which states whether calling it can suspend the current process (this is not allowed in the interrupt context)
- Locking - tools for synchronising access between multiple threads. This contains:
	- Spinlocks - one process locks a resource, then if another tries to claim the lock that process will continue using CPU cycles until the lock is released, instead of suspending and waiting for it to unlock. This is used in cases such as interrupts
	- RW locks - these allow concurrent access for readers, but exclusive access to writers, as many readers can read the same file without effecting each other, but only one file can write at a time
	- Mutexes - if a process tries to claim a locked resource, it is put to sleep and the CPU is given a new thread to run
	- Atomic variables - a variable which is architecturally guaranteed to be atomically updated, i.e. in one go without any other process interfering. This is useful to avoid needing to take locks when only one variable is being changed, e.g. a counter that is shared across threads
- `current` and `get_current()` - pointer to the calling task descriptor ( a `struct task_struct` which contains the program name (`comm`), [[W7N1 - OS concepts#^260d9e|process ID]] (`pid`), and more)
- lists - a unified definition for doubly linked lists