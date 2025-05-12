A digital system that works with anything more complex than literal TRUE FALSE values has to encode its inputs and outputs into binary codes. The design of a code is dependent on what the code has to represent, as well as the logical structure and physical properties of the system.

# Keywords
- **Symbol**: The basic elements used to construct codewords.
- **Cardinality**: The number of different symbols available.
- **Codeword**: The number of different symbols available.
- **Bit/digit**: Each place in a codeword where a symbol may be put.
- **Encoding**: A mapping between a set of codewords and the set of data they represent.
- **Code**: A particular set of codewords and their encoding/decoding.
We will mostly be working with *fixed-length binary codes*, so cardinality 2 with symbols 1 and 0 and for each code all codewords have the same number of bits.

# Unary codewords
A codeword with cardinality 1 must have a different number of digits for each codeword.
E.g.

| Symbol | Value |
| ------ | ----- |
| \|     | 1     |
| \|\|   | 2     |
| \|\|\| | 3     |
This is not very useful, as circuitry would have to change size according to the value of information being represented, which though possible is inconvenient.

The quantity of information represented by a unary code grows linearly with the circuit size, meaning it would quickly require infeasibly large circuits.

# Binary codewords
A code with fixed length codewords and cardinality 2 can efficiently represent a large number of values

| No. of bits | Maximum values represented |
| ----------- | -------------------------- |
| 1           | 2                          |
| 2           | 4                          |
| 3           | 8                          |
| 4           | 16                         |
| $n$         | $2^n$                      |

In this case the amount of information represented grows exponentially with the length of the codewords, so a fairly small circuit can handle a large number of values.

# Types of codes
Examples:
![[w2FoodCooker.png]]
![[w2Thermometer.png]]
**Hamming distance** is the number of bits which differ between 2 codewords
![[w2Gray.png]]
![[w2sevenSeg.png]] ^6c2d76
### ASCII
7 bit code for representing graphical symbols
Has a more complex but deliberate structure

00\*\*\*\*\*: 32 non-graphic symbols
0000000 and 1111111 mean 'no symbol'
**10**0001 = 'C'
**11**0001 = 'c'
Bit 6 = is character
Bit 5 = is lowercase if it is a character
**001**0000 = '0', **001**0001 = '1', ..., **001**1001 = '9'
Numerics all start 001

This means that characters are can be alphabetically ordered by comparing the last 5 bits, numbers the last 4