**FPGAs** are [[W8N1 - Fabrics#Synthesis of physical circuits|reprogrammable]] logic devices, where both the physical logic and interconnect can be reprogrammed after manufacture.
# Physical structure
## Layers
![[w8n2FpgaUpperLayer.png]]
The upper layer in an FPGA consists of thousands of identical logic cells and interconnects that are all programmable by the lower layer.

![[w8n2FpgaLowerLayer.png]]
The lower layer consists of a set of [[W5N3 - Dynamic Random Access Memory (DRAM)|DRAM]] cells, one cell for each logic cell and one for each interconnect [[W5N6 - Multiplexer|mux]].
These DRAM cells are then programmed to set the behaviour of the FPGA.

## LUTs
The logic cells are made of [[W5N4 - Read Only Memory#Combinational logic using ROM|LUTs]] (in this case using a slightly more complex circuit):
![[w8n2FpgaLut.png]]
In this case this is a 2 input LUT.
The values $P_{3:0}$ are set in the logic cell's corresponding memory cell.
This then produces the truth table:

| A\\B | 0     | 1     |
| ---- | ----- | ----- |
| 0    | $P_3$ | $P_1$ |
| 1    | $P_2$ | $P_0$ |

This means that all 16 possible 2 bit logic functions can be defined using 1 standard logic cell.

E.g.
$P_{3:0}=1110$

| A\\B | 0   | 1   |
| ---- | --- | --- |
| 0    | 1   | 1   |
| 1    | 1   | 0   |

Gives this truth table, which is equivalent to A NAND B

$P_{3:0}=0111$

| A\\B | 0   | 1   |
| ---- | --- | --- |
| 0    | 0   | 1   |
| 1    | 1   | 1   |

Gives this truth table, which is equivalent to A OR B

$P_{3:0}=0110$

| A\\B | 0   | 1   |
| ---- | --- | --- |
| 0    | 0   | 1   |
| 1    | 1   | 0   |

Gives this truth table, which is equivalent to A XOR B

## The logic cell
![[w8n2FpgaLogicCell.png]]
By configuring memory bits $P_{j,k,l,m,n}$ it is possible to change the various MUXs to reroute the input and output signals around the cell, and with the included [[W5N1 - D flip-flop|D flip-flop]] allowing for persistent values or [[W7N1 - Finite state machines|FSMs]] to be implemented. `INIT` can be used to initialise the D-FF to a value, such as for initialising an FSM state. The P and C on the D-FF stand for preset and clear, and will set the D-FF to 1 or clear it to 0.

### Possible configurations
![[w8n2LogicCellConfigurations.png]]
There are many configurations:
1. $P=010**$ is a simple combinatorial function
2. $P=*01**$ is a flip-flop
3. $P=011**$ is a [[W7N2 - FSM Classification#Class-1 FSM|class-1 FSM]]
4. $P=*00**$ directly passes A to output
5. $P=111**$ is a [[W7N2 - FSM Classification#Class-3 FSM|class-3 FSM]]

## Interconnect
![[w8n2Interconnect.png]]
An switch allows a vertical line to be connected to a horizontal line via a grid of switches (programmed in the same way as a logic cell).
Some critical signals like the clock have dedicated interconnect lines.
There are only so many interconnect lines, and trying to pass too many signals through a small region of the chip leads to routing congestion or failure.
We can use logic cells as pass-through or change parameters in the synthesis process to get a different implementation.

## Combining cells
A single logic cell is more complex than a NAND or D-FF, but is still too small for a whole solution to most non-trivial problems.
Most commercial cells have no more than 6 inputs, 2 outputs, and 1 or 2 D-FFs.
Therefore, multiple cells must be combined:
![[w8n28InputAnd.png]]
Using many logic cells, arbitrary functions can be created, such as the above design for an 8 input AND.

Signal routing is as important as logic placement in the construction of a good implementation.

The place-and-route process, part of synthesis, is extremely computationally costly and is impossible to determine the optimum solution for any non-trivial layout, so designer knowledge can have a significant impact on viability and performance.

# FPGAs vs [[W8N1 - Fabrics#Custom gate ASIC|ASICs]]
## FPGA
- Combinational functions:
	- If below the number of inputs for the LUT then complexity is free, but once over the input count at least 3 cells are needed and extra interconnect
- Flip-flops:
	- Cheap if used as there is 1 in each cell, expensive if not used (there is 1 in each cell)
- Interconnect:
	- Cheap up to threshold density, then very expensive (either using cells as wire or leaving them unused)
- Possible to reflash an FPGA in the field meaning shorter time to market as all faults don't have to be caught pre-production
## ASIC
- Combinational functions:
	- Cost scales with number used
- Flip-flops:
	- More expensive than combinatorial functions, though cost still scales with number used
- Interconnect:
	- Can trade off interconnect and logic density, allowing for finding the optimum balance between the 2
- An error found post-fabrication are extremely costly, requiring either entirely new production runs or extra circuitry on PCBs to account for the error, meaning there is a longer time to market as more complete testing is required.
