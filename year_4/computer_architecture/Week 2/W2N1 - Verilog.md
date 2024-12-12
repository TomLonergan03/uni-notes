Verilog is a hardware description language that can be used to describe a system at 3 levels:
- Gate level: by instantiating AND/NAND/NOR etc. gates
- Synthesisable subset of behavioural Verilog for high level designs
- Non-synthesisable behavioural Verilog for testbench simulations
![[w2n1verilogDesignFlow.png]]
# Values
The fundamental value in Verilog is a 1-bit value with 4 logic values: 0, 1, x (undefined), z (not driven). Values are transmitted via nets (essentially a wire), and are stored in registers. Values are combined using operators to create expressions, which can be assigned to nets or to registers.
# Nets
A net represents a connection, with the most common type being `wire`. The default type of an `output` is a `wire`. Nets cannot store values, just propagate them.
```verilog
module AND (x, y, z);
	input x, y;
	output z;
	wire z; // not required, z is a wire by default
	assign z = x & y;
endmodule
```
Any change to `x` or `y` appears at `z` immediately, and the `assign` statement is known as continuous assignment.
# Registers
A `reg` type can retain a value that is assigned in a process. It can declare storage elements, such as flip-flops or memory, or hold variables that have no storage.

The `always @*` means that if any of the inputs change, the process will be rerun. `always @(posedge ck)` means that the process will be rerun on each positive edge of the signal `ck`.
```verilog
module mux(
	input a,
	input b,
	input c,
	output reg q
);

always @*
	if (c == 1’b1)
		q = a;
	else
		q = b;
endmodule
```
Here, `q` has no storage, as it gets reassigned every time either input changes. This means it is simplified to a `wire`.

```verilog
module D_LATCH (
	input d,
	input ck,
	output reg q
);

always @*
	if (ck == 1’b1)
		q = d;
endmodule
```
Here, `q` is a latch, which will store the result until a new value of `d` is present alongside a high on `ck`. We generally don't want to make latches, and should instead do this:
```verilog
module EDGE_D (
	input d,
	input ck,
	output reg q
);

always @(posedge ck)
	q <= d;
endmodule
```
Here, `q` is a clocked flip-flop, as each positive edge of the `ck` captures the value of `d`.
# Vectors
We often want to work with vectors of wires or registers. These are defined using `[upper:lower]`, with `upper` $\geq$ `lower`. `lower` can be any integer, not only 0. Verilog will fill in missing vector elements with zeros at the upper end if it expects more elements than it sees.
```verilog
reg [31:0] data;
reg [31:16] upper_half;
reg [15:0] lower_half;

always @(data)
begin
	lower_half = data[15:0];
	upper_half = data[31:0];
end
```
# Literals
`0`, `1`, `x`, `z` represent themselves. Literals are usually sized with a prefix of `size'base`. `x` or `z` sets one bit for `'b`, 3 bits for `'o`, and 4 bits for `'h`. If the most significant bit is `0`, `x`, or `z`, it will be automatically extended to fill the remaining bits above that position.
```verilog
wire [31:0] data1, [15:0] data2;
wire enable;
tri [31:0] bus; // tri declares a tri-state wire, i.e. can be [0,1,z]
assign enable = 1’b0; // ’b = binary
assign data1 = 32’h0000_0000; // ’h = hex (underscore is ignored, helps visually)
assign data2 = 16’d256; // ’d = decimal
assign bus = 32’bz; // assigns z to all 32 bits of the bus
assign data2 = 16’b0101 // assigns 0000000000000101 to data2
```
# Vector concatenation
`{vector1, vector2}` concatenates vectors, and `{n {vector}}` repeatedly concatenates `{vector}` `n` times.
```verilog
wire [31:0] data1, [15:0] data2, [3:0] data3;
assign data3 = 4’b0011;
assign data2 = {4{data3}}; // assigns 0011001100110011 to data2
assign data1 = {16’d0, data2}; // assigns 00000000000000000011001100110011 to data1
// N.B. 16’d0 == {16{1’b0}}
```
# Numbers
Verilog supports integer and real data types. Operators on integers and reals may not be mapped to gate level logic during synthesis (Vivado can do +-\*/ for `integer`, but not for `real`). There is a special ordinal type called `time` that can store simulation times.
```verilog
real epsilon;
real pi;
integer ipi;
time sim_time;

initial
begin
	epsilon = 4e-10;
	pi = 3.141592653589793238462643383279;
	ipi = pi; // will be rounded down to 3
	sim_time = $time; // $time always yields simulation time
	end
```
`initial` is not synthesisable, and runs once at the start of a simulation.
# Arrays
Arrays can be created from `reg`, `integer`, `time`, and `vector` types, but not from `real`.
```verilog
reg [7:0] memory [0:1023]; // 1K bytes of memory
reg [9:0] addr;
reg [7:0] dout;
reg [7:0] din;

always @*
begin
	dout = memory[addr]; // read from memory at address addr
end

always @(posedge clk)
begin
	if (write_enable == 1’b1) // write only if the enable signal is set
		memory[addr] <= din; // write to memory at address addr
```
# Reduction operators
Reduction operators reduce a vector to a single bit:
- AND: `&expr`
- OR: `|expr`
- XOR: `^expr`
```verilog
reg [3:0] x, y;
reg z1, z2, z3, z4, z5;

initial
begin
	x = 4’b1100;
	y = 4’b0111;
	
	z1 = &x; // z1 gets (1 & 1 & 0 & 0) = 0
	z2 = &x[3:2]; // z2 gets (1 & 1) = 1
	z3 = |y; // z3 gets (0 | 1 | 1 | 1) = 1
	z4 = ^x; // z4 gets (1 ^ 1 ^ 0 ^ 0) = 0
	z5 = ~^y; // z5 gets ~(0 ^ 1 ^ 1 ^ 1) = ~(1) = 0
end
```
# Modules
A module is the building block from which hierarchical circuits are composed, and can be instantiated within higher-level modules. Modules communicate via ports, with each port being a named signal or vector of signals. Input ports must be driven by connecting them to regs or wires in the calling module, while output ports **must drive wires only**.
```verilog
module foo (x, y);
	input wire x;
	output wire y;

	assign y = ~x;
endmodule

module bar (a, b);
	input wire a;
	output wire b;

	foo foo1(a, b); // positional args
	foo foo2(.x(a), .y(b)); // named args
endmodule
```
Module ports can be inputs (`input`), outputs (`output`), or bidirectional (`inout`) (though these are rarely used).
```verilog
module foo (x, y, z);
	input wire [6:0] x; // vector of input wires
	output wire [13:0] y; // vector of output wires
	inout tri z; // tri-state input/output

	assign y = {x, {7{z}};
	assign z = y[0] ? 1’bz : x[0]; // drive z with x[0] if y[0] is true
endmodule
// P.S. Can you see a problem with the circuit above?
// y[0] = z, so y[0] ? 1’bz : x[0] won't work as expected
```