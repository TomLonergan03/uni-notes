# Flip-flop timing parameters
![[w5n3flipflopTiming.png]]
There are several key timing parameters for flip-flops:
- **Clock period ($t_{cp}$)**: the nominal delay between successive positive clock edges
- **Clock uncertainty ($t_u$)**: the maximum potential jitter with respect to $t_{cp}$ from one positive edge to the next
- **Setup time ($t_{su}$)**: inputs to a flip-flop must be stable for a certain period before the clock posedge
- **Hold time ($t_{hld}$)**: inputs to a flip-flop must be stable for a certain period after the clock posedge
- **Clock to Q delay ($t_{cq}$)**: delay from clock posedge to a new output from the flip flop
# Clock skew
**Clock skew** ($t_{skew}$) is the time difference between posedges reaching different components, caused by the fact that there is not actually one single clock signal, but instead a **clock tree** of buffers between a component and the actual clock.
# Slack
![[w5n3slack.png]]
$$
\begin{aligned}
\text{Setup slack}&=t_{cp}+t_{skew}-t_u-t_{cq}(FF1)-t_{logic}-t_{su}(FF2)\\
\text{Hold slack}&=t_{cq}(ff1)+t_{logic}-t_{hld}(FF2)-t_{skew}
\end{aligned}
$$

**Setup slack** ($t_{su\_slack}$) is the extra time between a signal stabilising at a flip-flop and the next posedge occurring, e.g. 5ns slack means the signal is ready 5ns before it is sampled, while a -1ns slack means the signal is ready 1ns *after* it is sampled, which can result in incorrect behaviour.
**Hold slack** ($t_{hld\_slack}$) is the extra time between a signal being sampled and it next changing.

Both setup and hold constraints must be $\geq0$. The setup constraint limits the depth of logic paths for a given clock frequency, whereas violating the hold constraint will be a functional failure no matter what clock frequency is set to.
## Derivation of slack equations
### Setup
![[w5n3setupSlackDerivation.png]]
The path from $1\rightarrow4$ is the delay from the $CLK_{IN}$ posedge to $D2$ changing. $D2$ must be stable by point $8$. Point $8$ is calculated by the path $1\rightarrow5\rightarrow6\rightarrow7\rightarrow8$. The time from $1\rightarrow4=t_{T1}+t_{cq}+t_{logic}$ and the time from $1\rightarrow8=t_{cp}+t_{T2}-t_u-t_{su}$, so 
$$
\begin{aligned}
t_{su\_slack}&=\text{Point }8-\text{Point }4\\
&=t_{cp}+t_{T2}-t_u-t_{su}-(t_{T1}+t_{cq}+t_{logic})\\
&=t_{cp}+t_{skew}-t_u-t_{cq}-t_{logic}-t_{su}\\
\\
\text{where}\ \ t_{skew}&=t_{T2}-t_{T1}
\end{aligned}
$$
### Hold
![[w5n3holdSlackDerivation.png]]The path from $1\rightarrow4$ is again the delay from the $CLK_{IN}$ posedge to $D2$ changing. The path from $1\rightarrow9\rightarrow10$ is the time in which $D2$ cannot change. The time from $1\rightarrow4=t_{T1}+t_{cq}+t_{logic}$ and the time from $1\rightarrow10=t_{T2}+t_{hld}$, so
$$
\begin{aligned}
t_{hld\_slack}&=\text{Point }4-\text{Point }10\\
&=t_{T1}+t_{cq}+t_{logic}-(t_{T2}+t_{hld})\\
&=t_{cq}+t_{logic}-t_{hld}-t_{skew}\\
\\
\text{where}\ \ t_{skew}&=t_{T2}-t_{T1}
\end{aligned}
$$
# Impact of process, voltage, and temperature (PVT)
Logic delays are dependent on voltage (V), temperature (T), and process (P). V and T are environmental conditions, and can be varied after manufacturing, while P is set at manufacture, and can be somewhat random within a run. Manufacturers will often **bin** chips - sort them by performance after manufacturing and sell the same design as several different models depending on how fast they ended up.

High voltage, low temperature, and fast process conditions all result in faster logic. The best case (BC) is a chip at maximum voltage, minimum temperature, and fastest process, while the worst case (WC) is at minimum voltage, maximum temperature, and slowest process.

Setup and hold constraints must be checked under different sets of conditions, with the WC conditions used to determine setup slack and the BC conditions used to determine hold slack.
# Metastability
Flip-flops are bi-stable devices, meaning they are stable in states 0 and 1. If the setup and hold constraints are met, then the FF will always transition cleanly, but if they are not met an FF can enter a metastable state, where it is neither 0 nor 1. It will eventually stabilise to 0 or 1, but this may take some time.
![[w5n4metastability.png]]
# Synchronisers
Synchronous systems will still often have to work with asynchronous input signals, so those signals must be synchronised with the clock to prevent metastability occurring.
![[w5n4synchroniser.png]]
By using two fast settling flip-flops in sequence, even if $Q_1$ exhibits metastable behaviour, there is still a full clock cycle for it to settle before it is captured by $FF2$.

Synchronisation introduces an additional delay of at least one clock cycle, specifically:
$$
\begin{aligned}
\text{Maximum delay}&=2t_{cp}-t_{hld}+t_{cq}\\
\text{Minimum delay}&=t_{cp}+t_{su}+t_{cq}
\end{aligned}
$$
![[w5n4synchroniserDelays.png]]

Synchronisers are not perfect, and just reduce mean time between failures (MTBF), often from seconds to years or decades.
# Clock domains
Large digital systems typically have multiple interconnected subsystems, with each running off of a different clock. Each distinct clock defines a distinct **clock domain**.
![[w5n4clockDomains.png]]
The boundary between two clock domains is known as a **clock domain crossing (CDC)**, and outputs from that domain are synchronised to that domain's clock, so when two different clock domains interact, the signal must be synchronised by the receiving clock domain.
# System resets
Flip-flops must normally be reset on power up, as they will start in an unknown state, and may be reset in other cases as well. Resets may be synchronous or asynchronous, with synchronous resets requiring an AND gate on data input, which affects timing of normal reg-reg paths. Asynchronous resets feed into the asynchronous clear input on flip-flops, so do not affect the critical path of a design.
Asynchronous resets can cause metastability. On assertion of a reset, as long as the reset lasts a cycle or two, the metastability will settle out without impacting anything. The deassertion of reset must be synchronous however, as from then the system must function correctly.
![[w5n4resetSynchroniser.png]]
## Reset domains
A system may also have many **reset domains**. and there may be multiple reset domains within a clock domain, each with their own reset signal. **Reset domain crossings (RDCs)** must also be synchronised, or gate signals that cross the RDC with signals that produce `1'b0` during resets, or by disabling the clock for the receiving side of the RDC for the period in which the reset signal is active.