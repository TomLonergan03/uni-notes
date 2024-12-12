A buffer overflow can occur when a program reads an input into a buffer, ***without*** checking that the input fits within the buffer. If this buffer is allocated on the [[W8N2 - Memory management#^facff0|stack]], then the attacker may be able to craft an input which overwrites the current stack frame's return address, and therefore jump execution to any other part of the program, or to malicious code that was also inputted either as a part of the same buffer overflow, or via some other method.

# Shellcode injection
Often, an attacker will attempt to spawn a shell, which gives them general access to the system.
A shellcode attack will may something like this
```c
#include "stdio.h"

void main()
{
char *name[2];
name[0] = "/bin/sh";
name[1] = NULL;
execve(name[0], name, NULL);
}
```
Note:
- This must be compiled, then the resulting machine code injected
- The code must not contain any 0 bytes, as `printf`/`gets`/`strcpy` will stop copying on a null byte

# Finding the return address
In order to hijack the stack return address, we must know where the return address is relative to the start of the buffer, to know how much padding we require in order to overwrite it
![[w8n3ControlHijacking.png]]
There are a few approaches which can make this easier:
1. If we have access to the code, we can see how many variables of which sizes are allocated on the stack between the stack frame being created, and the buffer being created
2. If the code is not accessible, it can be possible to brute force it, as there are only $2^{32}$ possible locations in 32 bit memory space, and as the stack grows from the bottom of memory and is often limited to a few KB there are likely far fewer realistic options
3. By adding a large number of NOP (No Operation) instructions, before the start of the malicious code there is a greater chance that the return address is within that space, and once execution begins it will do nothing through the NOPs then reach the injected code (as shown above)

# Buffer overrun opportunities
There are many unsafe libc functions which manipulate buffers without checking the bounds of the buffers they manipulate, e.g.
- `strcpy(char *dest, const char *src)`
- `strcat(char *dest, const char *src)`
- `gets(char *s)`
- `scanf(const char *format, ...)`
All of these, and more, terminate on a null byte in `src`/`stdin`, regardless of if it overruns the end of the destination buffer.

# Arithmetic overflow exploits
Simply checking the size of the buffers is not always enough, for example, this function:
```c
int catvars(char *buf1, char *buf2, 
			unsigned int len1, unsigned int len2)
{
	char mybuf[256];
	if ((len1 + len2) > 256)
	{
		return -1
	}
	memcpy(mybuf, buf1, len1);
	memcpy(mybuf + len1, buf2, len2);
	do_some_stuff(mybuf);
	return 0;
}
```
is still vulnerable, as if we choose an appropriately high value for `len2` then `len1 + len2` can overflow and end up with a value that is less than 256.

# Format string exploits
`printf` has many exploits, often centred around unsafe usage where a string provided by the user is directly printed (`printf(str);`) instead of using a format string (`printf("%s", str);`). As the format string function call looks at the previous elements of the stack in order to find the arguments, we can have it print the stack using `printf(str);` where `str = '%08x.%08x.%08x.%08x.%08x|%s|';`.