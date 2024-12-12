# Source code to execution
1. The **compiler** converts C code to assembly code
2. The **assembler** converts assembly code to machine code
3. The **linker** links the binary to dependencies and libraries
4. The **loader** sets up address space in memory, loads the machine code into that address space, and jumps to the first instruction of the program
5. The **CPU** interprets instructions:
	- `%eip` is the extended instruction pointer, which points to the next instruction to be executed
	- `%eip` is incremented after each instruction, and can be modified by `call`, `ret`, `jmp`, and conditional `jmp`

# x86-32 registers
The x86 architecture includes the general purpose registers `%eax`, `%ebx`, `%ecx`, `%edx`, `%edi`, and `%esi`. Most instructions perform operations on these registers.
There is also the extended stack pointer `%esp`, which points to the top of the stack, and the extended base pointer `%ebp`, which points to the base of the stack frame for the current function call.
![[w7n1ProcessMemoryLayout.png]]

Each function call pushes a stack frame to the stack in the form: ^facff0

|                                                |
| ---------------------------------------------- |
| arguments in reverse order                     |
| return address                                 |
| stack base pointer of the previous stack frame |
| exception handlers                             |
| local variables                                |

# Summary
## Calling function
1. Pushes arguments onto the stack in reverse
2. Pushes the return address to the stack
3. Jumps to the functions address
## Called function
4. Pushes the old frame pointer onto the stack
5. Sets the frame pointer to the current end of the stack
6. Pushes local variables onto the stack
## Returning function
7. Resets the stack pointer to the current base pointer, and resets the base pointer to the previous base pointer
8. Jumps back to the return address