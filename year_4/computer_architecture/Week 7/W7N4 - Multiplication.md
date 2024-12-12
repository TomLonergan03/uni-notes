# Bit-serial multiplication
**Bit-serial multiplication** works using a shift register, gating each shifted multiplicand with 1 bit of the multiplier, and summing the results. E.g. $1011\times0101$:
$$
\begin{aligned}
1001\times0101&=\ \ \ \ \ \ 1001&\times1\ \ \ &\text{least significant bit}\\
&+\ \ \ \ 10110&\times0\ \ \ \\
&+\ \ 101100&\times1\ \ \ \\
&+1011000&\times0\ \ \ &\text{most significant bit}\\
&=0110111
\end{aligned}
$$
This multiplication can be implemented using hardware, or can be carried out using software on processors which do not have a hardware multiplier.
![[w7n4bitSerialMultiplication.png]]
Here, the multiplicand is passed to the adder only when the least significant bit being sent to the control is a $1$.
This can be optimised to only need a 32-bit adder by noticing that as the product register fills, the multiplier empties at the same rate.
![[w7n4optimisedBitSerial.png]]
Therefore, we can combine the product and multiplier in one 64-bit register, and shift it each time.
## Signed multiplication
For 2s complement, the MSB has a negative coefficient so it must be subtracted instead of added if the MSB of the multiplier is 1, or alternatively multiply the 31 bit operands and negate the product if the operand signs differ.
# Booth's algorithm
As $6=-2+8$, so too does $0110=-0010+1000$. This means that there are fewer additions needed to multiply, only one addition or subtraction each time a sequence of 1s is followed by a zero or vice versa.

E.g. if $A=120_{10}=0001111000_2$ then there is a $0\rightarrow1$ boundary at bit 3 and a $1\rightarrow0$ boundary at bit 7, so $A=2^7-2^3$. Hence, $A\times B=(2^7-2^3)\times b=-2^3\times B+2^7\times B$. This reduces the 10 possible additions to one addition and one subtraction.
# Carry-save adders
![[w7n4carrySaveAdder.png]]
A **carry-save adder (CSA)** produces the results of summing each bit pair, and all carry values separately. This can then be summed using one final [[W7N3 - Adders|carry-propagate adder]] to get the overall result. Using CSAs allow for many results to be added in parallel, which is the case for summing multiplication partial products.
# Wallace trees
For a $n\text{ bit }\times m\text{ bit}$ product there are $m$ partial products, with up to $n$ significant digits, we can produce a Wallace tree. If we consider all single bit partial products we get $P_{i,j}=a_i\cdot b_j\cdot2^{i+j}$. The $2^{i+j}$ scaling factor is known as its $weight=i+j$.
![[w7n4wallaceTree.png]]
## Wallace tree construction
We have $k$ partial sums each level, with level $d_j$ being the $j$th level of the tree. $k\leq d_j$, with $d_0=2$ and $d_{j+1}=\lfloor3d_j/2\rfloor$. We start with a table of partial sum values $P_{i,j}$ which are grouped every 3 rows:
![[w7n4wallaceTreeInitial.png]]
Then in each group of 3 rows, we combine groups of three at the same weight into 2 signals using a compressor, and groups of 2 at the same weight into one signal. Any ungrouped signals are passed down to the next level, giving us these results on level 3:
![[w7n4level4results.png]]
Here, a black dot is the result of the previous sum, with the line indicates where the carry propagates to, and a white dot being the value from the previous level. The first 3 rows from level 4 form the first 2 rows at level 3, and rows 4-6 on level 4 form rows 3 and 4 at level 3. These are grouped into 3 and combined again:
![[w7n4wallaceTreeLevel3.png]]And are propagated and combined again for level 2:
![[w7n4wallaceTreeLevel2.png]]
And again for level 1:
![[w7n4wallaceTreeLevel1.png]]
And level 0 combines everything using a conventional carry propagate adder:
![[w7n4wallaceTreeLevel0.png]]
## Signed multiplication
For signed multiplication, a negative multiplicand must be extended across the entire summation: $S\ S\ S\ S\ S\ S\ x_4\ x_3\ x_2\ x_1\ x_0=0\ 0\ 0\ 0\ 0\ (-S)\ x_4\ x_3\ x_2\ x_1\ x_0$. To represent the value $-S$ in a column we complement the original sign bit, and add 1 at that position, eliminating the sign extension digits and reducing the complexity of the Wallace Tree.
## Dadda Trees
![[w7n4daddaTree.png]]
By only performing the minimum required compression on each level to allow for the next level, we can reduce the amount of logic required in early levels. We can also delay using partial product bits that can be inverted (as part of signed multiplication) until later in the tree, which reduces the critical path length.
# Radix-4 multiplication
The previous multipliers have been radix 2 multipliers, which use 1 multiplicand per bit of the multiplier. Radix 4 multipliers split an $n$ bit multiplier into $n/2$ overlapping groups of 3 bits, centred on positions $i=0,2,4,...,n-2$ (for an even $n$) and uses a modified version of Booth encoding.
E.g. $n=8$ gives four 3 bit groups:
![[w7n4radix4multiplication.png]]

There are 8 possible values for each group:

| Multiplier bits<br>($x_{i+1},x_i,x_{i-1}$) | Comment                       | Booth coding interpretation | Partial multiplier | Sign of partial multiplier |
| ------------------------------------------ | ----------------------------- | --------------------------- | ------------------ | -------------------------- |
| $0\ 0\ 0$                                  | String of 0s                  | $0$                         | $+0$               | $+$                        |
| $0\ 0\ 1$                                  | End of string of 1s           | $1$                         | $+1$               | $+$                        |
| $0\ 1\ 0$                                  | Start and end of string of 1s | $(-1)+(+1)\cdot2$           | $+1$               | $+$                        |
| $0\ 1\ 1$                                  | End of string of 1s           | $(+1)\cdot2$                | $+2$               | $+$                        |
| $1\ 0\ 0$                                  | Start of string of 1s         | $(-1)\cdot2$                | $-2$               | $-$                        |
| $1\ 0\ 1$                                  | End and start of string of 1s | $(+1)+(-1)\cdot2$           | $-1$               | $-$                        |
| $1\ 1\ 0$                                  | Start of string of 1s         | $-1$                        | $-1$               | $-$                        |
| $1\ 1\ 1$                                  | Middle of string of 1s        | $0$                         | $-0$               | $-$                        |
All of the operations on the partial multipliers are simple to do ($0$ means exclude, $+1$ means include, $+2$ means include and shift left 1, $-2$ means complement and shift left, $-1$ means complement).

(Radix 8 is possible, but results in partial multipliers that aren't powers of 2, so is much less useful)

