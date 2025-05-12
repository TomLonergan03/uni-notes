![[w10n3FsmLoop.png]]
Most behaviours in [[W7N1 - Finite state machines|FSMs]] are in the form of loops, where every state is (eventually) reachable from every other state. However, all sparse state codes, and most dense state codes, have at least a few unused state codewords. If these show up in the state register, then it will either trigger unintended behaviour or cause a fault detection system to go off. Additionally, [[W7N1 - Finite state machines#Modular state machines|modular state machines]] require multiple different machines to be in corresponding states, then the correspondence can be maintained via exchanging signals.

The start state of a [[W5N1 - D flip-flop|D flip-flop]] is not known, so an FSM that is simply initialised by powering $V_{DD}$ will begin with a random codeword. This will often lead to either non-existent state codewords or skipping initialisation states.

# When and what gets initialised
Initialisation may be required after power is applied and before the system begins operation, or sometimes after power is lost and recovered, or to end a phase of active operation, or after an invalid state is detected.

Different kinds of initialisation may be required:
Total loss of state can be undesired for the sake of debugging after a fault is detected. Alternatively, it can be desirable for security reasons, blocking a malicious actor from retrieving data from a system that is not in use.

[[W7N3 - ASM Diagrams|ASM chart language]] is not capable of describing asynchronous events like initialisation, so we must explicitly make the link between fabric level initialisation signals and circuits, and the desired initial state.

The alternative to initialisation is to ensure the FSM always behaves correctly for any possible initial codeword, by either using all states and having a design that can start in an state, or by having a transition from all unused states to the desired initial state.