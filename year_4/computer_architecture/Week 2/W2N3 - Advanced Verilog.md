# Processes
Logic circuits do not evaluate sequentially like a program, instead all hardware components are simultaneously active. Verilog represents these parallel activities using processes. A process has a header and a body: `always <header> <body>`.
- The header defines a "sensitivity list": the list of signals that triggers the process to run
- The body defines the logic to be evaluated when any signal in the sensitivity list changes
 
Verilog supports combinational and sequential processes:
- Combinational process are triggered by any change in their inputs, so we say they are sensitive to any change in the level of those inputs
- Sequential processes are triggered when a specified signal value transition occurs, e.g. the rising edge of a clock
```verilog
module NAND (
	input x,
	input y,
	output reg z);

always @*
	z = ~(x & y);
endmodule
```
`@*` specifies an implicit sensitivity list

```verilog
module clock_gen (
	output reg clk);

initial
	clk = 1’b0;

always #10
	clk = ~clk;
endmodule
```
`initial` triggers at the start of the simulation, `#10` triggers every 10 units of time

```verilog
// If items are missing from
// the sensitivity list, the
// logic will not simulate
// correctly.
always @(x)
	z = ~(x & y);
```
It's best to avoid explicit sensitivity lists
# Assignment operators
There are two assignment operators:
- Non-blocking: `state_r <= state_nxt;`
	- Used in synchronous processes
	- All non-blocking assignments occur simultaneously when the trigger fires
	- All right-hand expressions are evaluated, and then all are assigned to the left hand side reg variables
- Blocking: `state_nxt=3'd2;`
	- Used in combinational processes
# Behavioural statements
All the following statements can only be used within a process:
## Conditionals
```verilog
// begin..end not needed for single
// statements
always @*
	q = (a > b) ? a : b;

// also usable on RHS of ‘assign’
assign q = (a > b) ? a : b;

// begin..end is needed for multiple
// statements
// ‘if’ statement must be always
// inside a process
always @*
	if (c == 1’b1)
		begin
			o1 = 1’b0;
			o2 = 1’b0;
		end
	else
		begin
			o1 = d1;
			o2 = d2;
		end
```
## Case
Similar to switch in C/C++/Java/etc. Matches from top to bottom.
```verilog
// Basic form of case statement
case (expression)
	option1 : statement1;
	option2 : statement2;
	option3 : statement3;
	default : default_statement;
endcase

// Case with shared statements
case (expression)
	option1 : statement1;
	option2, option3 : statement2;
	default : default_statement;
endcase
```
We can also use `casez` if we don't care about certain values:
```verilog
// 4:2 priority encoder implemented with a case statement
//
// This is an efficient way to express logical intent,
// which logic synthesizers are able to optimize well.
reg [3:0] requests;
reg [1:0] selected;

casez (requests)
	4’b1???: selected = 2’b11;
	4’b01??: selected = 2’b10;
	4’b001?: selected = 2’b01;
	4’b0001: selected = 2’b00;
endcase
```
We should never use `x` or `z` in the patterns, as synthesis and simulation likely won't agree.
# Compiler directives
Macros are defined using \`
```verilog
`define INCLUDE 1 // define a macro

`ifdef `INCLUDE
`include my_header.v // include other file
`endif

`timescale 1ns / 10ps // reference time unit 1ns, precision (minimum time unit) 10ps
// values must be 1, 10, or 100
```
# Testbench constructs
There are certain constructs that are used only in testbench simulations, to simplify testing.
## File handling
```verilog
// file_handle = $fopen(string);
integer handle1;

initial
	begin
		handle1 = $fopen(“outfile.txt”);
		#50000 begin
			$fclose(handle1);
			$finish;
		end
	
		forever #5 clk = ~clk; // 100 MHz
	end
	
// $fdisplay(handle, output);
// $fwrite(handle, output);
// $fmonitor(handle, output);

$fdisplay(handle1, “Hello world”); // print to console
$fmonitor(handle1, $time, “a = %d”, a); // timestamped print
$fwrite(handle1, “a = “, a); // write to file
```
## Reading into arrays
This is typically used to load hex data into an array, and is typically used within an `initial` process.
```verilog
// $readmemb(“<filename>”, memory_name);
// $readmemb(“<filename>”, memory_name [, start [, end]]);

reg [7:0] memory [0:1023];
integer addr;

initial
	begin
		$readmemb(“memory.dat”, memory);
		$display(“Contents of memory”);
		for (addr = 0; addr < 1024; addr = addr + 1)
			$displayb(memory[addr]);
	end
```
# Example: 7-segment display driver
```verilog
// digit0 and digit1 have 7 bits.
// Each bit turns on one
// segment of a 7-segment display.
//
//    -0-
// 1 |   | 5
//    -6-
// 2 |   | 4
//    -3-
//
// Given a 4-bit switch setting,
// representing a numerical value,
// define the on/off values for
// each of the two digit displays.

reg [6:0] digit0;
reg [6:0] digit1;

wire [3:0] switches;

always @*
begin
	digit1 = 7’b0000000; // no leading zero
	digit0 = 7’b0111111; // zero by default
	case (switches)
		4’b0001: digit0 = 7’b0110000; // one
		4’b0010: digit0 = 7’b1101101; // two
		:
		4’b1010: digit1 = 7’b0110000; // ten
		4’b1011: begin
			digit0 = 7’b0110000; // -one
			digit1 = 7’b0110000; // one-
		end
		:
		:
	endcase
end
```