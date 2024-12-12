Instead of executing each instruction as cycle (with a lower clock speed) pipelining divides instructions into a number of phases.
![[w3n3pipelinePhases.png]]
- **FCH**: fetch instruction from memory and increment PC
- **DEC**: decode instruction and fetch values from general purpose registers
- **EXE**: execute arithmetic/logic operations or address computation
- **MEM**: memory access or branch completion
- **WRB**: write back results to general purpose registers
# Fetch
![[w3n3fetch.png]]
First, an instruction is read from memory at the address given by the program counter, either from instruction **closely-coupled memory (ICCM)** or the **instruction cache (I-cache)**, and the program counter is incremented to point to the next instruction.
# Decode
![[w3n3decode.png]]
[[W3N2 - Instruction set architectures#RISC-V base 32 bit instruction format|Instructions]] may have:
- 2 source registers (R/S/B-type)
- An immediate (I/S/B/U/J)
The first ALU operand is normally `xreg(rs1)`, but may be PC for branches or `AUIPC`. The second ALU operand may be `xreg(rs2)` or an immediate.
# Execution
![[w3n3execute.png]]
Many instructions involve an ALU operation, with the result passed to the MEM stage.
# Memory
RISC-V load (I-type) and store (S-type) instructions use `rs1` + immediate to calculate the memory address in the EXE stage, which is passed to the MEM stage for reading or writing.
# RISC-V M extensions
![[w3n3riscvMExtension.png]]
The M extension supports multiply and divide instructions, which require additional hardware to support. A 32x32 multiply can be completed in 1 cycle, while a divide is typically multi-cycle so the instruction would remain in EXE for several cycles
# Writeback
![[w3n3writeback.png]]
The instruction results are committed to the architectural state when the instruction reaches the writeback stage
# Branching
![[w3n3fullPipeline.png]]
Branch tests are calculated independently from the ALU, allowing the ALU to calculate the branch target address in parallel with the test condition. If a branch is taken, then the next fetched instruction is no longer correct so the pipeline is flushed and fetching restarts from the target address.
# RISC-V function calls and returns
The `JAL` (jump and link) and `JALR` (jump and link register) instructions allow jumps with the address of the instruction following the jump written to a register.
![[w3n3jal.png]]
![[w3n3jalr.png]]
# Store operations
![[w3n3store.png]]
![[w3n3storeKey.png]]
Store operations cannot write to memory until after they have committed, so a **store queue (STQ)** stores pending writes until after the store instruction reaches the writeback stage, and also until EXE doesn't contain a load instruction, and the STQ never needs to have more than 2 entries (idk why)
# Representing pipeline timing

|               | Cycle 1 | Cycle 2 | Cycle 3 | Cycle 4 | Cycle 5 | Cycle 6 | Cycle 7 |
| ------------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- |
| Instruction 1 | IF      | DEC     | EXE     | MEM     | WB      |         |         |
| Instruction 2 |         | IF      | DEC     | EXE     | MEM     | WB      |         |
| Instruction 3 |         |         | IF      | DEC     | EXE     | MEM     | WB      |
# Pipeline balance
Each pipeline stage is a [[W2N2 - Boolean logic|combinational logic network]], and the longest circuit delay over all stages determines the clock period. Ideally, the delays through each pipeline stage is identical, but in practice this is hard to achieve.