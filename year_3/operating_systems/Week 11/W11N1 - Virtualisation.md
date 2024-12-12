**Hardware virtualisation** is the process of creating a virtual version of real hardware, which is capable of running a complete operating system and its applications.
![[w11n1virtualisation.png]]
A virtualised system has these components:
- the **virtual machine**: a virtual instance of a physical machine. This is known as a **guest machine**.
- the **host machine**: the physical hardware running one or more virtual machines.
- the **hypervisor** or **virtual machine monitor (VMM)**: management software which partitions the physical resources of the host between the various guests, and manages how and where they run
# Virtualisation requirements
- **Efficiency**: a VM should execute the majority of instructions directly on the host machine
- **Resource control**: the VMM must maintain full control of all resources
- **Equivalence**: a guest should behave indistinguishably from if it was running as a physical machine. (there are two exceptions here, one for runtime due to the overhead of virtualisation, and the other is resource availability, as it is controlled by the VMM)
# Benefits of virtualisation
- **Resource optimisation**: isolation allows multiple OSes to share the same hardware
- **Security**: the host machine is protected from the guest machines, and the guests from each other. A malicious or infected guest will not affect the host or other guests
- **Workload migration**: the VMM can move a running guest from one host to another with no interruptions or change in guest state. This allows load balancing and fault tolerance in datacenters
# Uses of virtualisation
- **Personal**
	- Running multiple OSes on one host without needing to reboot between them
- **Technical**
	- Operating system/hardware design/development
	- Kernel debugging/testing
- **Commercial**
	- Data centre server consolidation
	- Making servers fault-tolerant and able to migrate between hosts
# Types of VMM
## Type 0
A **type 0** or **firmware** VMM provides support for virtual machine creation and management via firmware, without software management. This was often used in mainframes and old servers.
## Type 1
A **type 1**/**native** VMM is software that runs directly on the physical machine in place of an OS, and is used in datacenters and servers.
## Type 2
A **type 2**/**hosted** VMM runs as an application inside an operating system, and runs virtual machines as individual [[W3N1 - Processes|processes]]. Examples include virtualbox, Parallels, and QEMU.

![[w11n1vmmTypes.png]]
# Physical resources
A VM still needs to use physical resources, such as memory, storage, networking, and graphics. The VMM shares out these resources so that VMs are unaware that they are being virtualised. The VMM emulates the hardware resource that the VMs expect, and some devices, such as dedicated network cards or GPUs, may be passed straight through to a specific VM.
# Methods of virtualisation
There are two main ways to virtualise a machine: 
- **binary translation**: the VMM examines each instruction the guest wishes to execute, and determines the result of it. This is very slow, but necessary for cross-architecture virtualisation (e.g. running an ARMv7 guest on an x86-64 host), and for virtualisation in cases there isn't hardware support for virtualisation on the host architecture.
- **trap-and-emulate**: the guest OS runs directly on the CPU in user mode and when it attempts to execute a privileged instruction it traps to the VMM. The VMM then emulates the attempted instruction, and returns the result along with control to the guest, who resumes execution.
# Paravirtualisation
**Paravirtualisation** is an enhanced form of virtualisation where the guest is aware it is being virtualised and cooperates with the VMM for optimal performance. This removes the need for trap-and-emulate, as the guest now directly requests privileged operations from the VMM.
# Migration
Guest machines can be migrated between hosts using one of two approaches:
- **Stop/restart**: the VM is stopped, its state is saved, and then loaded on another machine. This has a noticeable interrupt in the running of the guest
- **Live migration**: the VM's state is copied between hosts while the VM is running. The following process occurs:![[w11n1migration.png]]
	1. The source VMM establishes a connection with the target VMM and confirms it is able to send a guest over
	2. The target creates a new guest with a new vCPU, nested page table, and other state storage
	3. The source sends all read only pages to the target
	4. The source sends all read write pages to the target, and marks them as clean on the source.
	5. The source sends all dirty read write pages to the target, as some were modified while the initial transfer was occurring. This repeats until only very few pages are being sent
	6. The source freezes the guest and sends the final state the the target, which can now start running the guest
	7. Once the target confirms the guest is running the source terminates the guest
# Other types of virtualisation
- **Container virtualisation**: a container isolates a process or group of processes within a single OS, e.g. Docker or containerd
- **Language virtualisation**: a programming language runs on a simple virtual machine, e.g. the JVM. This allows a program to be compiled once for all possible platforms, as long as that platform has an implementation of the language's virtual machine.