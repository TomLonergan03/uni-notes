A code consisting of fixed length codewords can only represent a finite set of values. As most useful sets of numbers are infinite, most digital systems are unable to represent almost all values.
Integer codes tend to cover a range of integers completely, and not cover any value outside that range, except perhaps with a single special codeword.
Rational and pseudo-real number codes have unrepresentable numbers, even within a limited range. Codes are designed to ensure that holes in representation are put in places where they do the minimum possible harm.

A binary code representing a number is often not simply that number represented in base 2, and therefore we can not know what value is represented by e.g. 0101010 unless we know what value it represents.
Likewise, hexadecimal and octal notations are compact ways of writing binary codewords, but are not necessarily the actual number simply converted to base 16 or 8.

A number encoding has a **range** - the difference between the maximum and minimum value encodable - and a **resolution** - the distribution of coverage over that range

# Binary natural number (BNN)
A place-value exponential-weight code: the value of a 1 is determined by its location within the codeword, with each successive bit having an exponential increase in significance.

| bits      | $b_5$ | $b_4$ | $b_3$ | $b_2$ | $b_1$ | $b_0$ |
| --------- | ----- | ----- | ----- | ----- | ----- | ----- |
| weights   | $2^5$ | $2^4$ | $2^3$ | $2^2$ | $2^1$ | $2^0$ |
| codeword  | 1     | 0     | 1     | 0     | 1     | 0     |
| bit value | 32    | 0     | 8     | 0     | 2     |       |

Numeric value of codeword = $32+8+2 = 42$

A BNN encoding does not encode negative numbers

Range: $0\text{ to }(2^N-1)$ for $N$ bits
Resolution: uniform and unity

# Signed magnitude number (SM)
By adding a sign bit to a BNN code which multiplies the following number by $\pm1$.

| bits      | $b_6$  | $b_5$ | $b_4$ | $b_3$ | $b_2$ | $b_1$ | $b_0$ |
| --------- | ------ | ----- | ----- | ----- | ----- | ----- | ----- |
| weights   | $\pm1$ | $2^5$ | $2^4$ | $2^3$ | $2^2$ | $2^1$ | $2^0$ |
| codeword  | 0      | 1     | 0     | 1     | 0     | 1     | 0     |
| bit value | 1      | 32    | 0     | 8     | 0     | 2     | 0     |

Numeric value of codeword = $1\cdot(32+8+2) = 42$
Whereas $1101010$ has value = $-1\cdot(32+8+2)=-42$

Range: $-(2^{N-1}-1)\text{ to }+(2^{N-1}-1)$ for $N$ bits
Resolution: uniform and unity

# One's complement number (1sC)
Positive values are represented as in BNN with MSB false:
0010 = 2
0111 = 7

For negative values, invert the corresponding positive codeword:
1101 = -2
1000 = -7

Zero is either: 0000 (+0) or 1111 (-1)
Often only one is used, with the other representing no value or is automatically converted to the proper codeword

Range: $-(2^{N-1}-1)\text{ to }+(2^{N-1}-1)$ for $N$ bits
Resolution: uniform and unity

# Offset BNN codes
Add a constant positive bias to the value and encode biased value.

With a bias of +7:
0010 = 2 - 7 = -5
1101 = 13 - 7 = 6

Range: $-bias \text{ to } 2^N-1-bias$
Resolution: uniform and unity

# Two's complement number (2sC)
Like a BNN, but the most significant bit has negative weight

| bits      | $b_5$    | $b_4$ | $b_3$ | $b_2$ | $b_1$ | $b_0$ |
| --------- | -------- | ----- | ----- | ----- | ----- | ----- |
| weights   | $-(2^5)$ | $2^4$ | $2^3$ | $2^2$ | $2^1$ | $2^0$ |
| codeword  | 1        | 0     | 1     | 0     | 1     | 0     |
| bit value | -32      | 0     | 8     | 0     | 2     |       |

Numeric value of codeword = $-32+8+2=-22$

Range: Range: $-(2^{N-1})\text{ to }+(2^{N-1}-1)$ for $N$ bits (so is not zero balanced)
Resolution: uniform and unity

You perform subtraction by adding a negative 2sC number with a positive 2sC number using the same circuitry as for BNNs, however may have overflow (which can be ignored)

To convert between BNN and 2sC, simply invert the bits and add 1 
e.g. For -7, first we get the BNN for 7: 0111
Then invert it: 1000
Then add 1: 1001
This gives us: -8 + 1 = 7