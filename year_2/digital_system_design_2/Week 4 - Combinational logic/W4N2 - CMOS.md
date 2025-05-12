CMOS, or Complementary Metal Oxide Semiconductors, are the dominant integrated circuit technology. 
![[w4n2Cmos.png]]
There are 2 types of CMOS transistors:
- NMOS ^e7dd83
	- ON when G = HIGH
	- Poor conduction if A/B signals HIGH
	- Good conduction if A/B signals LOW
	- Used in pull-down networks
	- Smaller
- PMOS ^a66c68
	- ON when G = LOW
	- Good conduction if A/B signals HIGH
	- Poor conduction if A/B signals LOW
	- Used in pull-up networks
	- Larger

# CMOS inverter (NOT gate)
![[w4n2CmosNot.png]]
If $in$ HIGH then M1 off, M2 on, so out is LOW
If $in$ LOW then M1 on, M2 off, so out is HIGH

## Waveforms
![[w4n2CmosInverterWaveform.png]]
With an ideal input signal we can see that there is almost no power consumption for static signals whether HIGH or LOW, power is only consumed when changing.

# CMOS NAND gate
## NAND gate 1
![[w4n2CmosNand1.png]]
F is LOW only when A and B are HIGH

## NAND gate 2
![[w4n2CmosNand2.png]]
F is HIGH when either A or B or both are LOW

## NAND gate 3
![[w4n2CmosNand3.png]]
By combining gates 1 and 2, we have a NAND gate where $V_{DD}$ is never shorted to $0V$.

## n-input NAND gate
The CMOS NAND gate can be generalised to any number of inputs like so:
![[w4n2nInputCmosNand.png]]

# NOR gate
![[w4n2CmosNor.png]]
Similar to NAND but swapping NMOS and PMOS

## n-input
Again generalisable to n inputs as so:
![[w4n2nInputCmosNor.png]]

# CMOS AND gate
It isn't possible to implement AND using a simple pull-up and pull-down network so we simply chain together a NAND and a NOT gate
![[w4n2CmosAnd.png]]

# CMOS OR gate
Similar to AND, we must use NOT NOR instead of directly implementing an OR gate:
![[w4n2CmosOr.png]]

# Transistors per gate
| Gate | Transistors |
| ---- | ----------- |
| AND  | 6           |
| OR   | 6           |
| NAND | 4           |
| NOR  | 4           |
| NOT  | 2            |

This means it is more efficient to implement a boolean expression using NAND/NOR than AND/OR

# NAND vs NOR
PMOS transistors require more space for equal drive strength than NMOS transistors.
As NOR gates have PMOS transistors in series, while NAND gates have them in parallel, NAND gates are faster and generally smaller than NOR gates.

# General CMOS gates
A CMOS gate will have a pull-up network and pull-down network. These will contain an equal number of transistors, and the pull-up network will only be closed when the pull-down network is closed and vice versa.

## Important note
PMOS transistors cannot be used to connect to 0V
NMOS transistors cannot be used to connect to $V_{dd}$

# Constructing CMOS networks for a given Boolean expression
A Boolean expression must be in the form $F=\overline{SOP}$ where $SOP$ = Sum Of Products.
If the whole term has a bar over it, no inverter is needed at the output of the pull-up/pull-down network. If there isn't a bar, then an inverter is needed.
The pull-up/pull-down network is then derived from the inverse of the $SOP$

An example:
![[w4n2CmosEx1.png]]
![[w4n2CmosEx2.png]]
![[w4n2CmosEx3.png]]