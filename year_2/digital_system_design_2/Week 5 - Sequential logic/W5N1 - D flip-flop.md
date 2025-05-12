# Transmission gate
![[w5n1TransmissionGate.png]]
Allows signals when $G$ = HIGH, blocks them when $G$ = LOW

# D latch
![[year_4/computer_architecture/images/w5n1dLatch.png]]
A D latch is transparent ($Q=D$) when $G$ is HIGH, however on the falling edge stores value of $Q$.
![[w5n1DLatchBehaviour.png]]
$Q=D$ when $G$ is HIGH, and when $G$ falls $Q$ holds output.
$Q-$ is the previous value of $Q$ just before the most recent falling edge of $G$.

| $G$     | $D$ | $Q$  | Behaviour         |
| ------- | --- | ---- | ----------------- |
| 1       | 1   | 1    | Transparent       |
| 1       | 0   | 0    | Transparent       |
| Falling | *   | D    | D captured        |
| 0       | *   | $Q-$ | Output value held |

# D flip-flop
![[w5n1DFlipFlop.png]]
A D flip-flop stores a value ($D$) on the rising edge of the clock cycle, and outputs it ($Q$) and inverted ($\overline Q$).

![[w5n1DFlipFlopBehaviour.png]]
Samples $D$ on the rising edge of $Clock$ and outputs that sample on $Q$ until the next rising edge.
$Q-$ is the value sampled just before the most recent rising edge of $Clk$.

| $Clk$   | $D$ | $Q$  | Behaviour            |
| ------- | --- | ---- | -------------------- |
| 0       | *   | $Q-$ | Emit stored value    |
| 1       | *   | $Q-$ | Emit stored value    |
| Falling | *   | $Q-$ | Emit stored value    |
| Rising  | *   | D    | Input value captured |

# Sample window failure
![[w5n1SampleWindowFailure.png]]
For a window around the rising clock edge where IN must remain constant otherwise the sampling process may fail.
