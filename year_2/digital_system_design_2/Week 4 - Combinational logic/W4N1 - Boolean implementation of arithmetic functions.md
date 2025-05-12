# Combinational logic for binary addition
If we want to add 2 single bit numbers $A$ and $B$ then we can produce the truth table:

| $A$ | $B$ | Carry | Sum |
| --- | --- | ----- | --- |
| 0   | 0   | 0     | 0   |
| 0   | 1   | 0     | 1   |
| 1   | 0   | 0     | 1   |
| 1   | 1   | 1     | 0   |

So:
Carry = $A\cdot B$
Sum = $\overline A \cdot B + A\cdot \overline B=A\oplus B$

## Binary half adder
![[w4n1HalfAdder.png]]

# Full adder
If we want to add a multi-bit BNN, then each bit can be added using a half adder that is extended to take in a carry bit. This gives us the truth table:

| $A$ | $B$ | $Carry_{In}$ | $Carry_{Out}$ | $Sum$ |
| --- | --- | ------------ | ------------- | ----- |
| 0   | 0   | 0            | 0             | 0     |
| 0   | 0   | 1            | 0             | 1     |
| 0   | 1   | 0            | 0             | 1     |
| 0   | 1   | 1            | 1             | 0     |
| 1   | 0   | 0            | 0             | 1     |
| 1   | 0   | 1            | 1             | 0     |
| 1   | 1   | 0            | 1             | 0     |
| 1   | 1   | 1            | 1             | 1      |

Both $Carry_{Out}$ and $Sum$ have relatively complex formulae, so we can represent them using Karnaugh maps:
- For $Carry_{Out}$ we get
  ![[w4n1FullAdderCarryKarnaugh.png]]
  And therefore the Boolean expression: 
  $Carry_{Out}=A\cdot B+A\cdot Carry_{In}+B\cdot Carry_{In}$
- And for sum we get
  ![[w4n1FullAdderSumKarnaugh.png]]
  And therefore the boolean expression:
  $Sum=A\cdot \overline{B}\cdot \overline{Carry_{In}} + \overline{A}\cdot \overline{B}\cdot Carry_{In}+A\cdot B\cdot Carry_{In}+\overline{A}\cdot B\cdot \overline{Carry_{In}}$
  Which simplifies to
  $Sum=A\oplus B\oplus C$

This can be made into the circuit:
![[w4n1FullAdder.png]]

Or by combining 2 half adders and an OR gate:
![[w4n1FullHalfAdder2.png]]

# Ripple carry adder
![[w4n1RippleAdder.png]]
This will add an arbitrary length BNN code by chaining the carry bits of several full adders together

## Ripple carry adder delay
The best case delay of a ripple carry adder is the time it takes for 1 full adder to perform an addition, but this only occurs when no full adder outputs a carry bit e.g. doing 0101 + 1010

The worst case only occurs when a carry is generated at the LSB propagates all the way to the MSB where it is consumed, e.g. when doing 0101 + 1011

Therefore, if the propagation delay for a full adder from $Carry_{In}$ to $Carry_{Out}$ is $t_{carry}$, and the propagation delay from $Carry_{In}$ to $Sum$ is $t_{sum}$ then the worst case propagation delay of a ripple carry adder with $n$ adders is 
$$t_{adder}\approx (n-1)t_{carry}+t_{sum}$$
This means that a ripple carry adder with larger $n$ is slower than a smaller $n$ and that optimising $t_{carry}$ is much more important than $t_sum$

## Subtraction using a ripple carry adder
Subtraction can be performed by instead of doing $S=A-B$ instead doing $S=A+(-B)$ using [[W2N2 - Integer codes#Two's complement number (2sC)|2sC]]. 

## Overflow
There are 2 overflow cases:
- A negative number has another number subtracted from it and results in a number too low to be represented.
- A positive number has another number added to it, resulting in a number too high to be represented
In both cases this can be detected when the MSB changes.

We can detect overflow using the boolean equation:
$Overflow = Carry_{In}\oplus Carry_{Out}$

## Subtractor circuit
![[w4n1Subtractor.png]]
By inverting all bits on one number and setting initial carry bit to 1 the above circuit can subtract any 2 4 bit numbers and detect if overflow occurs.

# Adder/subtractor
##  1 bit
![[w4n11bitAdderSubtractor.png]]
By setting $bit_{invert}$ a full adder can be used for both addition and subtraction.

## n bit subtractor
The 1 bit adder/subtractor can be chained together as in ripple adders to produce an arbitrary length adder/subtractor
![[w4n1AdderSubtractor.png]]