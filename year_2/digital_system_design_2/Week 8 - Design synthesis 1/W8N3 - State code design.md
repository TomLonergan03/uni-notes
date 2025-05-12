When designing an [[W7N1 - Finite state machines|FSM]] we start with an [[W7N3 - ASM Diagrams|ASM chart]] describing behaviour, this is then translated to an [[W1N1 - Introduction to digital signals#Design Abstraction Layers|RTL]] structure that uses codeword flow, then this to a Boolean logic structure, then finally to transistor layout.
Looking at the RTL layer, we must create a set of codewords for state. Each state must have 1 unique codeword describing it.
![[w8n3Fsm.png]]
For example, we can make this code:

| State name | State codeword |
| ---------- | -------------- |
| EAGER      | 11             |
| GRUMPY     | 00             |
| DOZY       | 01             |
| ASLEEP     | 10             |

Each bit in the state codeword corresponds to 1 state register flip-flop.

# Impact of density on state codes
Density affects the number of flip-flops in a circuit, one FF per codeword bit.

Dense codes:
- Generally result in combinatorial logic with high fan-in (many inputs combined to make 1 output)
	- Can lead to poor [[W8N2 - FPGAs#The logic cell|FPGA logic cell]] utilisation, but also can give efficient combinational gate area use in an [[W8N1 - Fabrics#Custom gate ASIC|ASIC]]
- May reduce clock energy consumption in circuits with very high state counts as fewer FFs have to be driven by the clock
- Are cheaper for ASICs as it minimises the number of FFs used

Sparse codes:
- Can result in simple combinatorial functions, giving lower delays both in FPGAs and ASICs
- Can simplify the interconnect structure, especially important in FPGAs
- Often no more expensive than dense codes on FPGAs as FFs tend to be plentiful in FPGAs

# Design styles
## Unit distance
- Unit distance codes are often lower energy, as only a single FF is required to change on each transition. They also can have a natural sequence corresponding to the sequence of states.
- Exact unit distance for all transitions is not always easy to achieve, but also isn't essential, there are still benefits when some transitions are close to unit distance.
	- If exact unit distance is required, then can be achieved by duplicating states (multiple states with same behaviour but different codes) or adding linking states
- Can be designed for high or low density (e.g. [[W2N1 - Non-number codes#^6c2d76|thermometer vs gray codes]])

For this FSM:
![[w8n3FsmUnitDistance.png]]
with the code:

| State name | State codeword |
| ---------- | -------------- |
| EAGER      | 01             |
| GRUMPY     | 00             |
| DOZY       | 10             |
| ASLEEP     | 11             |

Here, each transition has a distance of 1, except `ASLEEP` to `GRUMPY`, which has a distance of 2.

## 1-Hot
One bit per state, with each codeword having 1 bit true and all others false, therefore is a sparse code.
As with all codes with unused codewords, an invalid codeword appearing in the state register must be addressed.
1-hot structure makes identification of an invalid codeword straightforward, so can be a good choice for high reliability applications.

![[w8n3Fsm1Hot.png]]
This FSM and the following code is an example of a 1-hot FSM.

| State name | $S_3$ | $S_2$ | $S_1$ | $S_0$ |
| ---------- | ----- | ----- | ----- | ----- |
| EAGER      | 0     | 0     | 0     | 1     |
| GRUMPY     | 0     | 0     | 1     | 0     |
| DOZY       | 0     | 1     | 0     | 0     |
| ASLEEP     | 1     | 0     | 0     | 0     |

## Direct State-Output Map
Each state codeword contains the output codeword.
If each output is unique then these can be used for the states, if not then extra bits are added to each codeword which distinguish between states.
This design means that all outputs are taken directly from flip-flops in the state register, so output signal delay or [[W10N1 - Timing considerations#Clock skew|skew]] can be tightly controlled.
Some applications require only a few states to be directly mapped.

If we map `RUN` to $S_0$ and `LOOK` to $S_1$ then we can produce the following FSM and codewords. $S_2$ is added to distinguish between GRUMPY and DOZY, as both have the same outputs.

![[w8n3FsmDirectMap.png]]

| State name | $S_2$ | $S_1$ | $S_0$ |
| ---------- | ----- | ----- | ----- |
| EAGER      | *     | 1     | 1     |
| GRUMPY     | 0     | 1     | 0     |
| DOZY       | 1     | 1     | 0     |
| ASLEEP     | *     | 0     | 0     |

The * represent a free choice of value decided in a later stage of synthesis, not that either option should be valid for the FSM.

## Summary
Each of these codes give the same behaviour as all are drawn from the same ASM diagram. Each will result in different circuits with different sizes, speeds, and energy consumption.

# Linking design views and processes
![[w8n3LinkingDesignViews.png]]
Constructing a **transition table** from the ASM chart link paths and the state code table is an automated process. It will produce something like this:
![[w8n3TransitionTable.png]]
Each field in the table represents 1 stage of the [[W1N1 - Introduction to digital signals#Design Abstraction Layers|RTL]] structure.

![[w8n3AsmToTransitionTable.png]]
A decision box represents a mapping between current state and input to next state and an output in a state box represents a mapping between current state and the output codewords.
Concatenated state and input forms input data, and next-state concatenated with output forms output data.

We can simplify the transition table by using * where inputs don't impact a transition:
![[w8n3TransitionTableSimplified.png]]
