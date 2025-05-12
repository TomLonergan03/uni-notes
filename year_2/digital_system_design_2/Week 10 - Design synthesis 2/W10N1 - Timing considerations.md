# Propagation delay prediction
Modelling delay at lower levels gives increasing amounts of precision at the cost of increasing computational demand.

- Fabrication process modelling: Model individual transistors
	- Used only for individual transistors or very simple gate structures
- Transistor level modelling
	- Used to model individual gates or critical gate networks
- Switch level simulation
	- Uses propagation delay calculated from earlier simulations to model gates as switches with specific delay
	- Simple enough for frequent resimulation, but should be pessimistic

## Propagation delay models for gates
![[w10n1PropagationDelayModel.png]]
Different levels of detailed models for a CMOS logic gate

In the switch + parasitic model, R_pmos and R_nmos are the resistance of a conducting [[W4N2 - CMOS#^a66c68|PMOS]] and [[W4N2 - CMOS#^e7dd83|NMOS]] transistors. C_gg is the capacitance of a MOS transistor gate, and C_dd is the capacitance of a MOS transistor channel.

The step response model is $V=V_{dd}(1-exp(-t/RC)$.
As a gate can be considered to finish switching when $V>0.5V_{dd}$ we can say that:
$$
\begin{aligned}
	0.5&=1-exp(-t_{PD}/RC)\\
	0.5&=exp(-t_{PD}/RC)\\
	\ln0.5&=-t_{PD}/RC\\
	-0.7&=-t_{PD}/RC\\
	t_{PD}&=0.7RC
\end{aligned}
$$
so we can model $t_{PD}$ as long as we know what $R$ and $C$ are (determined in more detailed sims).
As capacitance $C=C_{dd}+(N\cdot C_{gg})$ where $N =\text{ number of gates}$, increasing the number of gate inputs. This means that a 2-NAND gate will have lower propagation delay than an 8-NAND gate.

## Propagation delay models for interconnect
There are 3 interconnect models:
- On-chip short distance:
	- 0 resistance (or nearly)
	- Significant capacitance
		- Modelled by increasing capacitance of gate output so $t_{PD}$ of gate increases
- On-chip long distance:
	- Significant resistance and capacitance
	- Modelled using a separate interconnect parameter, again simplifiable to $t_{PD}=0.7RC$
- Off-chip long distance:
	- Significant resistance, capacitance, and inductance
	- Modelled using analogue modelling of resistor, capacitor, inductor circuits
	- Also has edge distortion and reflection effects which can be significant for clock signals

## Clock skew
**Clock skew** is the difference in arrival time of a clock edge at different components.
The absolute delay from clock source to a component is not normally an issue, however clock skew can be a major concern in medium and large systems.

![[w10n1ClockSkew.png]]
Here, if a rising edge is emitted at $t_0$, then it is quickly received by $t_1$, then hits each of $t_2,t_3,t_4$ until it eventually reaches $t_5$. $t_{CS}$ is the delay $t_5-t_1$ There are 2 issues here:
1. On interconnect A, the Q-FF5 signal has $t_{CS}$ less time to reach D-FF1 before the next clock edge. This is the same as extra interconnect propagation delay on A//
2. On interconnect B, the Q-FF1 signal may never be correctly caught by FF5 as the Q-FF1 data starts to change before the completion of D-FF5's hold time.

# Choosing a clock speed
A **synchronous design** is a design where every data signal starts at a flip-flop, passes through combinational logic/interconnect, and arrives at another flip-flop within 1 clock cycle.

The minimum clock signal period is the worst case combination of time values for getting any data signal from an FF output to the next FF input. The parameters are determined either by modelling or measurements.

# Example for [[W7N2 - FSM Classification#Class-2 FSM|class-2 FSM]]
![[w10n1Class2FsmClock.png]]
For this FSM, the critical path is from the state register, through the next state logic, and back to the state register.
For the critical path, $t_{PD-t}=t_{PD-sr}+t_{PD-is}+t_{PD-ns}$. 
The next-state codeword must be ready $t_{su}$ before the next clock edge, so $t_{ck}>T_{PD-t}+t_{su}$
And the clock skew must be assumed to start the first edge late and second edge early.
Therefore, the minimum clock period is:
$$t_{ck}=t_{PD-sr}+t_{PD-is}+t_{PD-ns}+t_{su}+t_{cs}$$

# Example for [[W7N1 - Finite state machines#Modular state machines|modular state machine]]
![[w10n1ModularFsmClockRate.png]]
There are now 2 critical paths, and we must have a clock period that guarantees both are completed within 1 clock cycle.
1. External delay path: $t_{PD-t1}=t_{PD-sr}+t_{PD-om1}+t_{PD-ie}+t_{PD-nse}$
2. Internal delay path: $t_{PD-t2}=t_{PD-sr}+t_{PD-is}+t_{PD-nsi}$
Therefore the minimum clock period is:
$$t_{ck}=MAX(t_{PD-t1}+t_{su2},t_{PD-t2}+t_{su2})+t_{cs}$$
The minimum clock period can then be inverted to calculate maximum frequency.

# Clock period and hold time
FF data input must remain stable for a minimum period $t_h$ after an active clock edge. This is not normally an issue, however, if propagation delay is very small or clock skew is large.
Hold time is not related to clock period, so changing clock speed does not affect hold time issues.

The way to fix hold-time issues is by reducing clock skew by adding delays to clock lines or rearranging components, or by adding extra delay to the problematic data signal.1