**Register transfer level (RTL)** design is a way of designing hardware systems, in which behaviour is expressed as a sequence of computations each taking place within a single clock cycle. Good RTL designs have a clean separation of control and data, with control logic often being a finite state machine, and the datapath involving arithmetic functions, temporary storage, and interfaces to receive input data and transmit output data.
![[w5n3rtl.png]]

# RTL operations
Any synchronous hardware design can be defined in terms of individual RTL operations. Each RTL operation moves data from one or more source registers, through a datapath made of combinational logic, and stores the result in a destination register. An RTL operation completes in one clock cycle, and the clock period is limited by the delay for a signal to pass through the longest combinational path, known as the **critical path**.
![[w5n3rtlOperations.png]]
# Good RTL coding style
- Only one clock per module, and should be first in the port list
- Never use a clock from a data signal
- Clear all state on reset with a common reset signal
- All flip-flops should be clocked on the posedge of the clock
- Never infer latches in a combinational always block
- One module per file
- Consistently register the inputs or outputs of each module
- Place all logic in combinatorial always blocks
- Place only assignments and if-else statements in synchronous always blocks