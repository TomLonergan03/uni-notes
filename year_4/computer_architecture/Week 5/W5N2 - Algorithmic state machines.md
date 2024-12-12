Algorithmic state machine charts can be used to design algorithms for implementation in hardware. It consists of 3 simple components:
1. States
   ![[w5n2asmStateBox.png]]
	- A state has a name, a binary code that represents it, and output signals/register operations
2. Decisions
   ![[w5n2asmDecisionBox.png]]
	- A decision box has a condition with a number of exit paths that are selected based on the result of the condition
3. Conditions
   ![[w5n2asmConditionalBox.png]]
	- Conditional boxes may or may not be evaluated. Register operations may define internal combinational logic, or may produce outputs.
# Examples
![[w5n2asmExamples.png]]
These show FSMs which start in state S_1 when the Reset_b signal is received, then check a flag which if set to 1 sets register R to 0. Then, register F is set to G, and state S_3 is reached, which is the final state.
# ASM blocks
An ASM block is a structure consisting of one state box and all the decision and conditional boxes connected to its exit path. An ASM block has one entry block and any number of exit points.
![[w5n2asmBlock.png]]
As each state is captured in [[W5N1 - Latches and flip-flops#D-type positive-edge-triggered flip-flop|flip-flops]] each ASM block evaluates based on the state captured at its entry point.
# ASM design example
We wish to design a digital system with two flip-flops, $E$ and $F$, and one 4-bit binary counter, $A$.
The individual flip-flops in $A$ are denoted by $A_4$, $A_3$, $A_2$, and $A_1$, with $A_4$ holding the most significant bit.
A start signal $S$ initiates the system by clearing the counter $A$ and flip-flop $F$. The counter is then incremented by 1 each clock cycle until the system stops.
Counter bits $A_3$ and $A_4$ determine the sequence of operations, if $A_3$ = 0 then $E$ is set to 0 and counting continues If $A_3$ = 1 then $E$ is set to 1, and if $A_4$ = 0 then the count continues, but if $A_4$ = 1, $F$ is set to 1 on the next clock cycle and the system stops counting.

This results in this system of 3 ASM blocks:
![[w5n2asmDesignExample.png]]
- The initial state $T_0$ is entered on reset, and it stays in that state until $S=1$, at which point $A$ and $F$ are cleared and it enters state $T_1$
- In state $T_1$, $A$ is incremented on each pulse. If $A_3$ = 0 then $E$ is set to 0, and if $A_3$ is 1 then $E$ is set to 1, and it checks the value of $A_4$ and returns to state $T_1$ if it is 0 or continues to state $T_2$ if $A_4$ is 1.
- In state $T_1$, $F$ is set to 1 and it returns to state $T_0$.

This can then be implemented in Verilog:
```verilog
module asm (
	input clk,
	input rst,
	input S,
	output reg F,
	output reg E,
	output reg [4:1] A
);

parameter T0 = 2’b00;
parameter T1 = 2’b01;
parameter T2 = 2’b10;

reg [1:0] state_r;
reg [1:0] state_nxt;
reg [4:1] A_nxt;
reg E_nxt;
reg F_nxt;

always @( posedge ck or posedge rst)
	if (rst == 1’b1) begin
		state_r <= T0;
		A <= 4’d0;
		E <= 1’b0;
		F <= 1’b0;
	end else begin
		state_r <= state_nxt;
		A <= A_nxt;
		E <= E_nxt;
		F <= F_nxt;
	end

always @* begin
	state_nxt = state_r;
	A_nxt = A;
	F_nxt = F;
	E_nxt = E;
	
	case(state_r)
		T0: if (S == 1’b1) begin
			state_nxt = T1; // goto T1
			A_nxt = 4’d0; // clear A
			F_nxt = 1’b0; // clear F
		end
		T1: casez (A[4:3])
			2’b?0: E_nxt = 1’b0; // clear E
			2’b01: E_nxt = 1’b1; // set E
			2’b11: begin
				E_nxt = 1’b1; // set E
				state_nxt = T2; // goto T2
			end
		endcase
		T2: begin
			F_nxt = 1’b1; // set F
			state_nxt = T0; // goto T0
		end
		default: state_nxt = T0; // impossible
	endcase
end
endmodule
```