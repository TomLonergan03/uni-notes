# Binary division
Unsigned binary division is performed in exactly the same way as we do manual long division, with each step delivering 1 bit of the quotient.
![[w9n2binaryDivision.png]]

![[w9n2binaryDivider.png]]

![[w9n2divisionAlgorithm.png]]
# Shift operations
Variable left and right shifts are common operations, which though it can be done iteratively is common enough for a single cycle solution to be desired. A **barrel shifter** does this by combining 1, 2, 4, 8, etc. bit shifters to be able to shift by any number of bits in one cycle
![[w9n2barrelShifter.png]]
Each layer of the shifter is a 2:1 mux, and the overall number of layers in the barrel shifter is equal to the maximum number of bits the value can be shifted by.
# Floating point operations
**Floating point (FP) numbers** approximate the set of real numbers, and provide a similar number of discrete values as we get with integers spread over a vastly larger range. A floating point number comprises of a sign, exponent, and mantissa. A value $F=-1^\text{sign}\cdot2^{\text{exponent}+\text{bias}}\cdot(1+\text{mantissa})$. The exponent $\text{bias}=2^{e-1}-1$ to allow for negative exponents.
![[w9n2floatingPointFormat.png]]
## Floating point addition and subtraction
![[w9n2floatingPointDatapath.png]]
- Mantissae must be aligned before they are added or subtracted. This is done by shifting the operand with the smaller exponent right until it aligns with the larger exponent
- The result may require a right shift to normalise it, and the exponent of the result must be adjusted accordingly
- The sign depends on the sign of the mantissa result and the sign of the operands
## Floating point divide
For FP divides, the exponents are subtracted, the mantissae are divided, and the result is normalised and rounded.
## Floating point multiply
For FP multiplies, the exponents are added and the exponent bias is subtracted, the mantissae are multiplied, and the result is normalised and rounded.
### Floating point fused multiply-add (FMA)
Multiply and add/sub operations typically take 3 or 4 cycles, and there are two very common numerical kernels that consist of a multiply followed by an add ($d[i]=a\cdot x[i]+y[i]$ is common in linear algebra and $y+=a[i]\cdot b[i]$ is common for CNNs and matrix multiplication). The FP multiplier result can be fed into the add logic without normalising and rounding in between, which saves cycles and retains full precision through the op, reducing the accumulation of rounding errors. A [[W7N4 - Multiplication#Wallace trees|Wallace tree]] or [[W7N4 - Multiplication#Dadda Trees|Dadda tree]] only needs one additional input to add an aligned mantissa to the product, and often has no additional time penalty. FMA can perform addition ($Y=A\cdot1.0+C$), subtraction ($Y=A\cdot-1.0+C$) and multiplication ($Y=A\cdot B+0.0$).
# Fractional fixed point numbers
Many digital signal processing (DSP) applications use fixed point numbers as they are simpler, lower power, and thus cheaper.
For an $n$-bit fixed point value, there are $f$ bits of fractional value and $i=n-f$ bits of integer value. The **unit of least precision (ULP)** is the minimum difference between values that is captured by that format, with $ULP=2^{-f}$. Fixed point Q-format numbers are denoted $i$Q$f$, and if $n=16$ we could have:
- 1Q15 with range \[-1 ...  +0.99996948\] and a ULP $= 3.05\cdot10^{-5}$
- 3Q13 with range \[-4 ... +3.99987793] and a ULP$=2.44\cdot10^{-4}$
## Fractional fixed point arithmetic
Addition and subtraction are performed in exactly the same way as for integer values, while multiplication requires scaling of the result.
Rounding is performed by adding 1/2 ULP to the result, then truncate it.
![[w9n2fixedPointRounding.png]]
# Single instruction multiple data
**Single instruction multiple data (SIMD)** operations operate on a vector of 8 or 16 bit values packed into a 32 or 64 bit word. This can be created by adding an extra bit into each adder, and inserting control bits into the add values.
![[w9n2simdAdder.png]]
If a XOR b = 1 then carry bits can propagate from bit 7 to bit 8. If a & b = 1 then a carry is generated into position 8, which is used when the upper adder performs subtraction.

![[w9n2simdAdderValues.png]]
This 35-bit SIMD adder is capable of operating on 1x32-bit scalar, 2x16-bit vectors, or 4x8-bit vectors.
