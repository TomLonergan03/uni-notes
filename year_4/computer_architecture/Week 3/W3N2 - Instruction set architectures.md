An **instruction set architecture (ISA)** defines the interface between software and hardware.
An ISA consists of a number of components:
- Operations
	- Operator functions (add, shift, xor, mul, etc.)
	- Data movement (load-word, store-byte, etc.)
	- Control transfer (branch, jump, call, return, etc.)
	- Privileged and miscellaneous instructions
- Operands (int32, int16, uint8, float32, etc.)
- Addressing modes (how we access data in regs/memory/etc.)
- Privilege levels (e.g. user, kernel, supervisor, machine)

This course will focus on the RISC-V ISA, a new open source ISA.

# ISA design considerations
When designing an ISA, there are many factors to consider:
- Complexity of target for compilers
- OS and programming language features
- Important data types (floating point, vectors, etc.)
- Size of instructions (typically 64/32/16/8 bits)
- Efficiency of execution
- Backwards compatibility may be desirable
- Support for extensions
All of these are trade-offs, and what solution is chosen depends on the goals of the ISA
## CISC vs RISC
- **Complex instruction set computer (CISC)**
	- Has many complex instructions, so easier to use when manually writing assembly programs
	- Supports higher level language features as single instructions
	- Small number of registers, so often has in memory operands
	- Maintaining backwards compatibility results in complexity growing over time
- **Reduced instruction set computer (RISC)**
	- Has fewer, simpler instructions, so an easier target for compilers
	- Large number of registers, so generally uses a load-store architecture
	- Simple and fast decoding, so uses fixed lengths and formats
## ISA guidelines
- **Regularity**: operations, data types, addressing modes, and registers should be independent
- **Primitive**: instruction sets should not attempt to match HLL constructs
- **Simplify trade-offs**: make it easy for the compiler to make choices based on estimated performance
# RISC-V base 32 bit instruction format
There are 6 formats for 32-bit RISC-V instructions:
![[w3n2riscvInstructionFormat.png]]
- **R-type** instructions provide 2 source registers and 1 destination register
- **I-type** instructions have 1 source register and an immediate, as well as a destination register
- **S-type** and **B-type** instructions have two source registers and an immediate, but no destination register
- **U-type** and **J-type** registers have one large immediate and a destination register
 In all cases, `opcode[1:0] == 2'b11` implies a 32-bit format, `rs1`, `rs2`, and `rd` are always in the same position, and the most significant bit of an immediate is always in bit 31 (which simplifies sign extension logic)