# Binary coded decimal numbers
By creating 10 unique binary "super-symbols" with 4 bits each, and each super-symbol represents a number between 0 and 9 (encoded using BNN or otherwise), leaving 6 unused codewords.

The number value is partitioned as base 10 place value, and encoded using super-symbols. If we use 4 bit BNN super-symbols, we get:

| $10^2$ | $10^1$ | $10^0$ |
| ------ | ------ | ------ |
| 3      | 2      | 9      |
| 0011   | 0010   | 1001   |

So 001100101001 = 329
This is difficult to use for mathematical operations, however by including a fixed or floating-point structure we can exactly represent all finite length base 10 numbers.
This is useful for calculations that must be exact.

# Fixed-point numbers
By using BNN with a fixed fraction point where 

| bits      | $b_5$ | $b_4$ | $b_3$ | $b_2$ | $b_1$        | $b_0$    |
| --------- | ----- | ----- | ----- | ----- | ------------ | -------- |
| weights   | $2^3$ | $2^2$ | $2^1$ | $2^0$ | $2^{-1}$     | $2^{-2}$ |
| codeword  | 1     | 0     | 1     | 0     | 1            | 0        |
| bit value | 8     | 0     | 2     | 0     | $\frac{1}{2}$ | 0        |

So $101010 = 8+2+\frac{1}{2}=10.5$

Range: $0\text{ to }+(2^{N-F}-2^{-F})$ for $N$ bits with $F$ fractional bits
Resolution: uniform, $2^{-F}$

Exact representation of 0.1, 0.2, 0.3, 0.4, 0.6, etc not possible using fixed-point BNN no matter the size.

# Floating-point numbers
A family of several similar codes defined in IEEE 754-1985.
All have three fields: sign, exponent, and significand/mantissa.

For the 32-bit base-2 (single-precision) format, the fields are 1:8:23 bits wide:
- Sign: 0 (+) or 1 (-) as in SM
- Exponent: 8-bit offset BNN with bias +127
- Significand: 24-bit fixed-point BNN code with 23 fraction bits. Only the fraction bits are stored, as the MSB is considered to always be 1.
The value of the code is computed as:
Numeric value = $(-1)^{sign}\cdot2^E\cdot1.significand$ 

Special codewords for infinity, tiny, NAN, underflow

Floating point cannot represent all real numbers but a subset of the rationals. The exponent provides large range at the cost of resolution. Resolution varies across the range. e.g. below 16,777,216 the resolution is 1, but it becomes 2 above it, meaning that only every even number above it is representable. The variable resolution can be written as $2^{-23}\cdot2^E=2^{E-23}$.