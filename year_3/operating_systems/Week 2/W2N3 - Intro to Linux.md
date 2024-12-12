Linux is a [[W1N2 - History of operating systems#Timesharing|multiuser]], [[W2N1 - OS Structure#Monolithic OS|monolithic OS]] designed around the concept of files - "Everything is a file" (pretty much). Linux can run and does on pretty much any hardware, from servers in datacenters to mobile phones to laptops to embedded systems.
# Interfaces
![[w2n3linuxInterfaces.png]]
# Kernel contents
There are many simplified diagrams of the kernel, e.g.
![[w2n3linuxKernelStructure1.png]]
or
![[w2n3linuxKernelStructure2.png]]
In reality, it looks more like this:
![[w2n3linuxKernelStructure3.png]]

The Linux kernel is also extensible, using [[W2N1 - OS Structure#^dd02e9|loadable kernel modules]], as well as using [[W2N1 - OS Structure#^5df907|runtime program injection]].

# Standard library
There is no way to write a [[W1N3 - Syscalls|syscall]] instruction in C. Instead, a C library is provided with one procedure per system call. It is the library interface - *not* the syscall interface - that is defined by POSIX.
POSIX specifies:
- which library procedures a conformant system must supply
- what their parameters are
- what they must do
- what results they must return

# Standard utility programs
Linux comes with a collection of standard programs, with around 150 of them being specified by POSIX. These programs include:
- File and directory manipulation commands
	- E.g. `cat`, `chmod`
- Filters
	- E.g. `grep`, `head`
- Process management tools
	- E.g. `ps`, `kill`
- Program development tools
	- Editors, compilers
- Text processing
- System administration tools
- Miscellaneous
