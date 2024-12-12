[[W8N3 - Buffer overruns|Buffer overruns]] are one of the most common cyberattack vectors. There are many ways to mitigate or prevent these kinds of attacks.
# Memory safe languages
Memory safe languages, e.g. Java, Python, Go, Rust, do things such as checking array bounds and pointer dereferences at compile time, and/or use garbage collection/ownership models remove the programmer from the error-prone task of managing memory. Whenever possible, use a memory safe programming language for new projects.
# Safe programming practices
Unsafe languages can still be used safely, either by using safe libraries or by manually validating bounds on each buffer/pointer access.

E.g. in C, using `size_t strlcpy(char *destination, const char *source, size_t size)` instead of `size_t strcpy(char *destination, const char *source)`.
# Code hardening
Code hardening techniques are compiling and operating system specific security measures which systematically defend against buffer overflow opportunities. These approaches are not perfect, but can make it much harder for an attacker to exploit an overflow opportunity.
## Stack canaries
A stack canary is a random value (that must be randomly generated on every execution) placed on the stack before the return pointer. This means that in the event of a buffer overflow, the canary will be rewritten whenever the return address is also rewritten.
![[w8n5StackCanary.png]]
### Limitations
Stack canaries can be defeated, in several ways:
- by learning the value of the canary, such as using a buffer overread
- by jumping over the canary, e.g. if the attack has a way to write to a section of memory that is not contiguous with the buffer
- by guessing the canary value - on 32 bit systems this is possible with enough time

Stack canaries also do not detect heap overruns
## Data execution protection - Write XOR Execute
We can generally divide memory into regions of code, which must executable, and regions containing data, which must be readable and writable. Therefore, we can make each region either executable or writable but not both.
The stack and heap are both writable, while the [[W7N1 - OS concepts#x86 process memory layout|text segment]] is executable.
This means that even if the attacker can get their malicious code on the stack/heap, it will still not be executed.
### Limitations
While data execution protection does present code injection, it is still usually possible to carry out a ***return-to-libc*** attack, as most C programs are linked against `libc`, which provides many useful calls for an attacker e.g. the `exec()` function executes the program passed as an argument. By jumping to `exec()` and falsifying the argument such that `exec` gets `/bin/sh` as its argument, the program will [[W8N3 - Buffer overruns#Shellcode injection|launch a shell]], giving access to most of the system.
![[w8n5returnToLibc.png]]
## Address space layout randomisation (ASLR)
Most operating systems support randomising the address space, which places each library, along with the stack and heap, at a different address that changes every time the program is executed. This means that an attacker cannot hardcode e.g. where `exec()` is, and would have to use some method to determine its location at run time. This is still not perfect however, and there are attacks that can get around it.

# Ultimately
Attackers have and will develop more complex methods of exploiting buffer overflows, and it is up to the programmer to
1. choose a memory safe language wherever possible
2. ensure that whenever user input is copied into a buffer that no more data is copied than that buffer can hold