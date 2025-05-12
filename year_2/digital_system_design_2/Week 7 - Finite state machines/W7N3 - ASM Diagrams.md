The representation of [[W7N1 - Finite state machines|FSMs]] purely on the [[W1N1 - Introduction to digital signals#Design Abstraction Layers|register layer]] makes it hard to tell what the actual state by state behaviour is representing.
![[w7n3FsmRtl.png]]
![[w7n3FsmCodeword.png]]
What exactly e.g. 0111 represents and when its outputs are produced is unclear.

We can't use a truth table, as the output of an FSM does not depend directly on its input, an FSM might take a constant input and give a constant output for a while, then without a change in input start producing a regularly changing output as it moves through a series of states internally.

# ASM Chart Language
An **ASM chart** is a formally defined language for constructing FSMs.

| Component       | Image                       | Definition                                                                                                                                                                                                                                                         |
| --------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| State box       | ![[w7n3StateBox.png]]       | A state, named in the oval and with fixed output signals in the box. An FSM is in a state box for one clock cycle (it may then transition back into itself).                                                                                                       |
| Decision box    | ![[w7n3DecisionBox.png]]    | A transition requirement, enters at the top, applies a Boolean expression, and exits on either TRUE/1 or FALSE/0 sides. It has no time delay. The order of decisions defines only the logical precedence of expressions, not the order in which they are compared. |
| Transition line | ![[w7n3TransitionLine.png]] | Lines may merge, but not split (splitting requires a decision box). Every loop must include at least 1 state box. It has no time delay                                                                                                                             |

# Initial and final states
![[w7n3FsmFinalState.png]]
A final state is unusual, as generally an FSM is intended to keep working indefinitely, but sometimes exists for certain failure cases or other design reasons. A final state is normally only escapable by cycling the power on the machine.

![[w7n3FsmInitialState.png]]
An initial state is more common, and is the state that an FSM starts in when powered on.

# Design process
1. Define logic inputs
   E.g. switch, temperature, numbers
   These affect internal state
2. Define logic outputs
   E.g. lamp, heater, numbers
   These are driven by internal state
3. Design internal behaviour
   Define names and sequences of internal states
4. Encode symbolic state names as state-register codewords
5. Synthesis: CAD tools translate FSM definition to physical circuitry

