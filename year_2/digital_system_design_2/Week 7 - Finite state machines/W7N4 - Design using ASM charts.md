![[w7n4TrafficLights.png]]
We will use the ASM [[W7N3 - ASM Diagrams#Design process|design process]] to design an FSM to control the traffic lights in this intersection.

The first step is to define inputs and outputs.
Inputs:
`TLS_NS` - `VEHICLE PRESENT ON NS => TRUE` 
`TLS_EW` - `VEHICLE PRESENT ON EW => TRUE` 

Outputs:
`NS_RED` - `TRUE => NS RED LAMP ON`
`NS_GREEN` - `TRUE => NS GREEN LAMP ON`
`EW_RED` - `TRUE => EW RED LAMP ON`
`EW_GREEN - TRUE => EW GREEN LAMP ON`

Then defining IO codewords and meaning:
![[w7n4TrafficLightCodewords.png]]
We have 2 normal codewords and 2 non-standard codewords, having both lights enabled at the same time is risky, as people may see the green and interpret it to mean go, whereas having neither is the safer failure mode.

The 2 codewords we will use are `STOP` and `PROCEED WITH CAUTION`.

# Class 2 FSM
We define a strict sequence of output codewords:
![[w7n4Class2Output.png]]
This will allow traffic on one road to run, then stops it, then allows the other road to run, then stops that.

We then build the state diagram to produce this sequence:
![[w7n4Class2Diagram.png]]
We have 4 states: `ALL_STP_A`, `EW_PWC`, `ALL_STP_B`, `NS_PWC`.
We have 2 `ALL_STP` states as each have different exit transitions, despite having the same outputs in both states.

A major disadvantage of this FSM is that it will stop traffic on one road even if the other is empty.

# Class 3 FSM
![[w7n4Class3v1.png]]
We could instead create this FSM if we incorporate inputs into the FSM.
This uses only 1 `ALL_STP` state, as the next state depends on which road has traffic on it.

This design, however, will always prioritise NS over EW (due to the order of decision boxes deciding priority). Therefore, if we have a constant flow of traffic on NS, EW will never get to go.

If we change the design to this:
![[w7n4Class3v2.png]]
We still have the same problem, except now EW will be prioritised over NS

This design fixes this by alternating priority between NS and EW on each clock cycle.
![[w7n4Class3v3.png]]
The problem with this is that it is starting to get unwieldy.

# Linked FSM
If we split it into 2 FSMs, one controlling the signals, and one deciding priority, we get this diagram:
![[w7n4LinkedFsm.png]]
This requires creating the `PRI_NS` signal, to transmit what is getting priority, as well as `ACT_NS` and `ACT_EW`, to transmit which road has most recently been green, between the FSMs.

The benefit of this design is that if we wanted to change the priority policy (e.g. perhaps NS has most of the traffic so should be prioritised 75% of the time) we just need to change FSM_B, whereas if we want to change the signals (e.g. adding an amber light) we just have to change FSM_A.

We then get the RTL structure:
![[w7n4LinkedFsmRtl.png]]

We have now produced a fairly complex system, and it also makes it apparent how this would have been much harder to design from the bottom up (starting with transistors) as was claimed with [[W1N1 - Introduction to digital signals#Design Abstraction Layers|design abstraction layers]].