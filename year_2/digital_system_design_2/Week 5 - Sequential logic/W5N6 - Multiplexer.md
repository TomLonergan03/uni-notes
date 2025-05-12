A multiplexer (mux) will select one of two or more logic inputs to output to a single logic output.
In general, an $n$ bit control signal selects from up to $2^n$ inputs.

# Example 2 input mux
![[w5n6Mux.png]]
This mux should output $A$ when $C$ is HIGH, and $B$ when $C$ is LOW. This has the Karnaugh map:
![[w5n6MuxKarnaugh.png]]
From which we produce the expression:
$$F=A\cdot C+B\cdot \overline{C}$$
Then we can produce the logic circuit:
![[w5n6MuxLogic.png]]

This is suboptimal, however, as [[W4N2 - CMOS#NAND gate 3|NAND]] and [[W4N2 - CMOS#NOR gate|NOR]] gates are the only gates directly implementable in transistors, and as [[W4N2 - CMOS#NAND vs NOR|NAND is better than NOR]], we can use [[W1N2 - Boolean algebra#Principles of Boolean algebra#Generalised De Morgan's|De Morgan's]] laws to recreate this in NAND only:
$$F=A\cdot C+B\cdot \overline{C}=\overline{\overline{A\cdot C}\cdot \overline{B\cdot \overline{C}}}$$
and produce this logic circuit:
![[w5n6MuxNANDLogic.png]]
(This uses 16 transistors vs the original circuit's 20 transistors, an approximate 20% reduction in size)

We could also build this out of [[W5N1 - D flip-flop#Transmission gate|transmission gates]], like so:
![[w5n6MuxTransmissionGate.png]]
However, although this gets us to 6 transistors, it also means that $F$ is no longer directly connected to $V_{DD}$, and therefore the drive strength is dependent on the drive strengths of $A$ and $B$, as well as the properties of the transmission gates.