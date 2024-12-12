Hazards are pipeline events that restrict the pipeline flow, in circumstances where two or more activities cannot proceed in parallel. There are 3 types of hazards:
- **Control hazards** arise from the pipelining of branch instructions, and others that change the PC (interrupts, exceptions, etc.)
- **Data hazards** occur when one instruction depends on the result of a previous instruction which has not yet exited the pipeline
- **Structural hazards** arise from resource conflicts where there is not enough of a resource to operate in parallel. These may be deliberate design choices, where the designers decide that specific structural hazards occurring would be an acceptable trade-off vs more complexity or performance impacts
# Structural hazards
## Memory ports
A simple example of a structural hazard is having only one memory port, meaning that if an instruction needs to access memory it will delay another instruction from executing instruction fetch by 1 cycle, as shown below

|               | Cycle 1 | Cycle 2 | Cycle 3 | Cycle 4 | Cycle 5 | Cycle 6 |
| ------------- | ------- | ------- | ------- | ------- | ------- | ------- |
| `lw $1,($2)`  | IF      | DEC     | EXE     | MEM     | WB      |         |
| Instruction 2 |         | IF      | DEC     | EXE     | MEM     | WB      |
| Instruction 3 |         |         | IF      | DEC     | EXE     | MEM     |
| Instruction 4 |         |         |         |         | IF      | EXE     |
## Multi-cycle operations
A multi-cycle divide unit may take 32 cycles to perform DIV or REM operations, which stalls all upstream stages for those 32 cycles. This has a significant impact on [[W1N1 - Principles of computer architecture#Factors that affect CPU performance|IPC]] of DIV and REM operations, but a small impact on the overall CPI of the processor, as DIV and REM ops are infrequent.

It may be possible to avoid blocking in some cases, such as with a 2 cycle multiply unit. This can be pipelined over the EXE and MEM stages, resulting in a CPI of 1 if the MPY result is not immediately used, or a CPI of 2 if the MPY result is used by the next instruction (which stalls at DEC for 1 cycle).
![[w4n1nonblockingMultiply.png]]
# Data hazards
![[w4n1dataHazard.png]]
Here, instructions 2, 3, and 4 are dependent on instruction 1, so must be delayed to start after cycle 5. This is called **stalling**.
![[w4n1dataHazardStall.png]]
## Data forwarding
If we bypass the register file, we can pull information from within the pipeline in certain situations, which reduces the number of stalls
![[w4n1dataForwarding.png]]

Data hazards involving use of load results can require a stall even if forwarding is implemented.
![[w4n1unavoidableStall.png]]

Code can be reordered at compile time, and if a compiler knows the pipeline structure for the target architecture it can rearrange instructions to minimise the number of stalls. This is called **code scheduling**.
![[w4n1codeScheduling.png]]
### Implementing forwarding in RISC-V
![[w4n1riscVForwarding.png]]
ALU results are available from EXE with no delay, LD results from MEM with 1-cycle delay, and all results from WRB. In all cases, we should use the youngest speculative result for each X-register.
# General performance impact of hazards
$$
\begin{aligned}
\text{Speedup from pipelining: }S&=\frac{CPI_\text{unpipelined}}{CPI_\text{pipelined}}\cdot\frac{\text{clock}_\text{unpipelined}}{\text{clock}_\text{pipeline}}\\\\

CPI_\text{pipelined}&=\text{ideal }CPI+\text{stall cycles per instruction}\\
&=1+\text{stall cycles per instruction}\\\\

CPI_\text{unpipelined}&\sim\text{pipeline depth}\\
\frac{\text{clock}_\text{unpipelined}}{\text{clock}_\text{pipelined}}&\sim1\\\\

S&=\frac{\text{pipeline depth}}{1+\text{stall cycles per instruction}}
\end{aligned}
$$
In practice, pipeline depth is usually between 5 and 12 stages, as it is difficult to split instructions into many tiny steps, and the overhead of excess stages can result in CPI loss for each extra stage.