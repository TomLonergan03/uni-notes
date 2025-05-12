A parity bit is an extra bit added to data in order to detect errors. The simplest forms are even/odd parity, where a bit is set to 0/1 to ensure there is an even/odd number of 1s in the data word.

## Example
3-bit data words with parity bit

| 3 bits of data | even parity | odd parity |
| -------------- | ----------- | ---------- |
| 000            | 0000        | 1000       |
| 101            | 0101        | 1101       |
| 111            | 1111        | 0111       |

We can produce the truth table for 3 bit parity:

| $B_0$ | $B_1$ | $B_2$ | $P_{odd}$ | $P_{even}$ |
| ----- | ----- | ----- | --------- | ---------- |
| 0     | 0     | 0     | 1         | 0          |
| 0     | 0     | 1     | 0         | 1          |
| 0     | 1     | 0     | 0         | 1          |
| 0     | 1     | 1     | 1         | 0          |
| 1     | 0     | 0     | 0         | 1          |
| 1     | 0     | 1     | 1         | 0          |
| 1     | 1     | 0     | 1         | 0          |
| 1     | 1     | 1     | 0         | 1          |

We then make the Karnaugh map for even parity:
![[w5n7ParityKarnaugh.png]]

As there is a checkerboard pattern, we can implement this using XOR:
$$P_{even}=B_0\oplus B_1\oplus B_2$$
And then produce the logic circuit:
![[w5n7ParityLogicCircuit.png]]

A single parity bit is the simplest form of error control, and there are many stronger codes that are used.