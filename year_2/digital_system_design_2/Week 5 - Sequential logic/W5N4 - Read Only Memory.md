- **Non-volatile** - memory content preserved over power off
- **Random access** - can be accessed in any order

There are several kinds of ROM, distinguished by when and if they are reprogrammable:
- **Mask ROM** - contents are programmed when the ROM is manufactured
- **PROM** - contents are programmed by blowing fusible links, this occurs after manufacture but is one time
- **EPROM** - contents can be electrically programmed, and erased using UV light
- **EEPROM** - contents can be programmed and erased electronically

An $n$-bit address is decoded to read one word in a ROM array.
Access tends to be fast (around 20ns)
![[w5n4ROMMemoryUnit.png]]

# ROM array
![[w5n4RomArray.png]]
When a $W$ is HIGH, then every $B$ line with an NMOS on that word line is 0 as $V_{DD}$ is connected to ground, while each $B$ line without an NMOS is a 1. The PMOS load transistors at the top prevent shorting $V_{DD}$ to ground from causing a current spike as PMOS transistors resist LOW signals, and as $V=IR$, a high $R$ means a low $I$ for a constant $V$.

# Combinational logic using ROM
As a ROM takes an input and gives the same output every time, an $n$ input $m$ output ROM can be used to implement any combinational logic function with $n$ inputs and $m$ outputs.
![[w5n4ROMLUT.png]]

This principle can also be used with [[W5N2 - Static Random Access Memory (SRAM)|SRAM]] or [[W5N3 - Dynamic Random Access Memory (DRAM)|DRAM]] to create the Lookup Tables (LUTs) used in FPGAs.