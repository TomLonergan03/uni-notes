# Scoreboarding
Scoreboarding is a dependency management system, which handles RAW, WAR, and WAW with proper stalls while allowing independent instructions to proceed.
There are 4 steps:
1. **Issue**: issue instructions to functional unit iff functional unit is free and no earlier instruction writes to the same destination register (prevents WAW)
2. **Read operands**: wait until source registers become available from earlier instructions through register file (prevents RAW)
3. **Execute**: execute instruction and notify scoreboard when done
4. **Write result**: wait until earlier instructions read operands before writing to register file (prevents WAR)
## Scoreboard organisation
The scoreboard keeps track of:
- Instruction status: which of the 4 steps each instruction operation is in
- Functional unit status: for each of the functional units the following is tracked:
	- Busy: if a functional unit is in use
	- Op: the type of operation to be performed
	- F$_i$: destination register
	- F$_j$ and F$_k$: source registers
	- Q$_j$ and Q$_k$: functional units producing F$_j$ and F$_k$
	- R$_j$ and R$_k$: ready flag, which indicates if F$_j$ and F$_k$ are ready but not yet read. Set to no after operands are read
- Register result status: indicates which functional unit will write to each register next, and reserves that register until the write occurs
## Conditions controlling instruction progression
- Issue condition: the functional unit is available and the result register is not the destination of another outstanding operation (prevents WAW)
- Read operands condition: both source registers are ready to read (R$_j$ and R$_k$ set to yes) (prevent RAW)
- Write register condition: no other functional unit is waiting to read the register that we want to read (prevent WAR)
## Scoreboard book keeping on instruction progression
- Issue actions:
	- Assign instruction to FU and mark it busy
	- Link source operands with RAW dependencies to their producing unit
	- Set ready flags if source register has no RAW hazard
	- Set the result producing ID of the destination register to be this FU
- Read operands actions:
	- Clear ready flags R$_j$ and R$_k$
- Write register actions:
	- Set any ready flags for all functional units depending on the current one
	- Clear pending result status for destination register of FU
	- Clear busy status of FU

# Limitations
- No [[W4N1 - Pipeline hazards#Data forwarding|forwarding]], but this is somewhat mitigated by having few stages after issuing
- Fairly regularly stalls at issue
- Out-of-order writes makes it difficult to have precise exceptions, as restarting execution is difficult if an exception occurs after a later instruction has written its result