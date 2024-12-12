By [[W3N3 - RISC-V pipelined instruction execution|pipelining]] instructions we can get CPI close to 1, but not all the way. [[W4N1 - Pipeline hazards#Data hazards|Data hazards]] are still an issue, and when an instruction stalls waiting for a dependency instructions behind it are also blocked until it resumes execution. Compiler scheduling can reduce the impact of hazards, but increases compiler complexity and has limited information, such as microarchitectural design and dynamically linked library usage. If we instead allow the processor to do **out-of-order execution**, it can then detect dependencies and rearrange instructions to minimise stalling, by allowing long running instructions to run alongside other instructions, e.g. a DIV may take many cycles, but only instructions that depend on its result need to wait that many cycles.
# Terminology
- **Instruction fetch**: fetch an instruction from memory
- **Instruction issue**: decode instruction, check for structural hazards, and send to execution units
- **Instruction execution**: execute instruction after registers are read once dependencies are cleared
- **Instruction completion/retire/commit**: finish instruction and update the processor state

There are some possible combinations:
- In-order issue, execution, and completion. This is not sufficiently parallel
- In-order issue, out-of-order execution, and in-order completion. This is the standard approach
- Out-of-order issue, execution, and completion. This is impractical to impossible.
# Data dependencies
There are 3 kinds of dependencies:
- **Read after write (RAW)** or flow dependencies
  ```
  MUL X3, X1, X2
  ADD X5, X3, X4
	```
	Here `ADD` depends on the result in `X3`
- **Write after read (WAR)** or anti dependencies
  ```
  MUL X3, X1, X2
  ADD X1, X5, X6
	```
	Here `ADD` can overwrite an existing value `MUL` depends on in `X3`
- **Write after write (WAW)** or output dependencies
  ```
  MUL X3, X1, X2
  ADD X3, X4, X5
	```
	Here `ADD` must complete after `MUL` or the value in `X3` will be incorrect
