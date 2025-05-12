Digital systems cover all systems that are constructed from **TRUE** and **FALSE**.

# Design Abstraction Layers

By designing in layers, starting from the most abstract and working towards increasing physicality we can solve a problem over time, whereas starting to build a microprocessor by designing individual transistor layouts would quickly become impossible.

|          Layer          |                      Purpose                      |                                      Precision                                       |
|:-----------------------:|:-------------------------------------------------:|:------------------------------------------------------------------------------------:|
|  Relational/functional  |    Identify and define features of the problem    |           Esimate likelyhood a solution exists within physical constraints           |
| Alogorithmic/procedural | Define large-scale features of possible solutions | Complexity choices strongly affect piming, performance and other physical properties |
|    Register transfer    |           Define hardware architecture            |                       Initial estimates of physical properties                       |
|          Gate           |        Define-small scale logic structure         |                        Good timing, power, and area esimates                         |
|     Physical layout     |    Micro-scale adjustments to critical regions    |        Timing, power, and area calculations are nearly exact but high-effort         |

# Making continuous signals discrete
Analogue circuit design concerns itself with continuous ranges of voltages and currents over a continuous range of time.
This is at odds with the binary system used in digital design, which instead uses 2 discrete values, of which only one can occur within a discrete unit of time.

This discrete behaviour can be produced by using nonlinear circuit behaviour, along with a clock that determines the discrete time.

![[w1l1TransferFunction.png]]
We can see that by using a non-linear transfer function, a larger range of the input voltage corresponds to a logic low or high in the output voltage. By maintaining a section of non-logic, we avoid noise passing through the gate and producing hysteresis.

A gate will not instantaneously transition between low and high, there will be a **rise** ($t_r$) or **fall time** ($t_f$):
![[w1l1RiseFall.png]]
During this period the output voltage is not a valid logic value, therefore systems must be designed to not use the output during these times.

A gate also has a **propagation delay** ($t_{PD}$), a delay before a change in input is reflected by a change in output. Propagation delay is measured between the half-way points of the two transitions:
![[w1l1PropagationDelay.png]]

# Representing logic signals
![[w1l1RepresentingSignals.png]]
We can abstract the noisiness of real world signals into straight lines, but still keep key features like transition time and propagation delay.

## Truth
There are 2 conventions for logic signals

### The positive logic convention
LOW voltage = FALSE
HIGH voltage = TRUE

### The negative logic convention
LOW voltage = TRUE
HIGH voltage = FALSE

It is standard to presume the positive logic convention, however some systems or sections of systems may use negative logic for some design reasons.

## Symbols representing logic values
We will use 1 to represent TRUE and 0 to represent FALSE; additionally * or X can be used to represent either value, where it doesn't matter or we don't know; and Z, which can be used to indicate an invalid logic state due to a logical or physical layer issue.

# The CLOCK signal
Below the register transfer layer, a digital circuit is working with continuous values, however in order to prevent this impacting the design of the circuit we must mask $t_{pd}$, $t_r$, $t_f$ etc. by discretising the time steps. 

This is done using a **CLOCK signal**: in its simplest form, it is a signal that alternates between the 2 logic levels at a constant rate.

The **clock cycle** is a single pair of consecutive high/low levels
The **clock period** is the unit of synchronous system time, or the time taken for one clock cycle.

# Sequential gates
![[w1CombinationalSequential.png]]
2 important blocks in the register transfer layer are:
1. *Combinational gates* which perform logic operations
2. *Sequential gates* which store logic values

## D flip-flop
![[w1DFlipFlop.png]]
D flip-flops are a basic sequential gate. 
Using a clock removes the variability in output delay as, as long as the delay is less than 1 clock cycle, it will always have updated by the time the next rising clock edge occurs.
A D flip-flop has a number of properties:
1. *Set-up time* is the critical period before a clock edge where the signal must not change.
2. *Hold time* is the critical period after an active clock edge where the signal must not change. The hold time is usually small, or even negative.
3. *Propagation delay* is the delay from active clock edge to the new stored value being transferred to OUT.
![[w1DFlipFlopShift.png]]
Here we can see that by the end of CLOCK cycle 1 DS1 is now outputting the same as IN was at the rising edge, then the brief HIGH on IN is ignored as it happens in the middle of a cycle. Finally, DS1 returns to LOW to match INs state on the second rising edge, as well as DS2 going to HIGH to match DS1 state on the rising edge.