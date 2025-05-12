There are 4 classes of [[W7N1 - Finite state machines|FSMs]]

# Class-1 FSM
In a class 1 FSM (also known as a delay machine or pipeline machine), codewords are fed forward only, there is no feedback where a later state can return to an earlier state.
![[w7n2Class1Fsm.png]]
This FSM will perform a series of operations on an input codeword, however the input codeword does not impact what happens to a later input codeword.

# Class-2 FSM
A class-2 FSM transitions once per clock cycle, with the next state always determined solely by the previous state. There are no conditional transitions.
![[w7n2Class2Fsm.png]]
This FSM will move 1 state at a time through a list of states, in a repeating pattern.

# Class-3 FSM
A class-3 FSM includes input into its next state logic, meaning that the next state transition is governed by both the current state and external systems.
![[w7n2Class3Fsm.png]]
This FSM will move into a new state based on the current state and the codeword provided through `INPUT`.

# Class-4 FSM
A class-4 FSM adds a path for external input to bypass the state register and directly control the output.
![[w7n2Class4Fsm.png]]
This FSM will either output based on current state, or based on current input, or some combination of both.