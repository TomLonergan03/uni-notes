A **finite state machine (FSM)** is a sequential logic machine which consists of a number of states, where some number of control signals are output, and a number of transitions, which govern what state transitions to what state depending on input signals.

![[w7n1StateRegister.png]]

# State register
The **state register** stores the codeword outputted by the FSM at the end of the previous clock cycle, i.e. it stores what the FSM output before beginning to transition to a new state.

# Modular state machines
We can join a number of smaller state machines together to create one large state machines.
For example:
![[w7n1ModularFsm.png]]
Here FSM A is a [[W7N2 - FSM Classification#Class-2 FSM|class-2 FSM]] while B, C, D, E are all [[W7N2 - FSM Classification#Class-3 FSM|class-3 FSMs]]. They all share a synchronised clock.