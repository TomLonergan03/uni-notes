# Review of [[W1N1 - Introduction to digital signals#Design Abstraction Layers|design layers]]
## Algorithmic
![[w8n1AsmChart.png]]
An ASM chart defines ideal algorithmic behaviour.
There is no concern here for voltage/propagation delay/setup time/etc.

## Register transfer
![[w8n1FsmRtl.png]]
First step of translating an algorithm to a physical design.
Define codewords, clock rates (also dependent on lower layers), IO formats etc.

## Gate
![[w8n1Gates.png]]
Deals with operations on logic functions on individual bits of codewords. More concerned with voltages, setup times, etc.
Many options, such as the pure 2 input NAND design above, or using purely NOR, or some other combination of gates.

## Physical layout
![[w8n1TransistorLayer.png]]
Now have total control over physical performance: area/power/speed/noise.
This does require a lot of effort to implement even simple logic functions, and total design of the original FSM is pretty much impossible at this level.
A lot of this work is automated, as translating gates into transistor implementations is both simple and labour-intensive.

# Synthesis of physical circuits
![[w8n1DesignSynthesis.png]]

Within a **fabric**, there are 2 components:
- Physical logic - gates/transistor grids
- Interconnect - wires connecting physical logic units

Each is either:
- Fixed - hardwired to one configuration
- Flexible - programmable (such as in FPGAs)

Different fabrics give different balances of logic/interconnect quantity and quality, of fixed/flexibility, and of technical performance options (energy, size, speed, reliability) and commercial options (cost, reliability, time-to-market).

# Examples of fabrics
## Custom gate ASIC
Designing a digital circuit with complete control over all aspects of design:
- Silicon substrate properties (limited by fabrication company)
- Transistor sizes, wire thickness, etc.
- NAND/NOR gate-level physical structure
- Block-level physical layout
- Interconnect physical layout
Total control of chip size, speed, energy tradeoffs possible.

Gives maximum performance at very high costs (an ASIC fab run often costs millions to get started and there is a massive quantity of design involved).
This cost is usually spread over a very high sales volume, or used in contexts where any cost is acceptable for obtaining maximum performance.

## Library cell ASIC
Designing a digital circuit using pre-designed library elements.
- Silicon, transistor sizes, wire thickness etc. all determined by library
- Gate physical structure limited to menu provided by library
- Block-level and interconnect layout controlled by designer/automation
Partial control over size/speed/energy tradeoff.

Gives good performance at high cost.
Usually requires a moderate sales volume to amortise high costs.

## FPGA
Uses a pre-existing chip, with capability to implement arbitrary logical functions.
- Physical fabrication already completed
- Gate structure fixed by chip
- Block-level and interconnect predefined but programmable
Being programmable and using existing hardware gives short time-to-market.
Size/speed/energy tradeoff is mostly fixed by choice of FPGA.

Worse performance than ASICs, however fairly cheap so a small sales volume covers commercial cost.

## Processors/microcontrollers
Uses a pre-existing chip with fixed logical functions.
- Fabrication already completed
- Logic and interconnect fixed by chip
- Programmable logic behaviour, with restrictions such as serialisation
Rapid design and short time-to-market.
Size/speed/energy tradeoff is fixed by chip choice.
Acceptable performance at moderate to very low cost (the cheapest microprocessors can be got for <10p per chip). Almost any sales volume can be accommodated.