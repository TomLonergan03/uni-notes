- **Static** - does not require refresh mechanism
- **Random access** - information can be accessed in any order
- **Memory** - stores bits for later use
- **Volatile** - memory content is lost when powered off
![[w5n2SRAMCell.png]]
Feedback between the 2 inverters stores a constant value.
When the $BL$ (Bit Line) side is HIGH, the cell is storing a 1, when the $\overline{BL}$ side is HIGH, the cell is 0.
$WL$ (Word Line) activates all cells on a row to be read or written.
Vs a [[W5N1 - D flip-flop#D latch|D latch]] uses 6 NMOS and 4 PMOS transistors, instead of 8 of each
Requires dedicated reading and writing hardware

# SRAM Array
![[w5n2SRAMArray.png]]
To read the lower word, the row decoder would output a 1 on $WL_0$ which would close the NMOS transistors which causes each of $BL_0$ or $\overline{BL_0}$ and $BL_1$ or $\overline{BL_1}$ to go to high. The sense amplifier then outputs the bits stored, as if a $BL$ is HIGH, then the cell it is attached to is storing 1, whereas if a $\overline{BL}$ is HIGH, then the cell is storing 0.
To write to the lower word, the row decoder would again output a 1 on $WL_0$. The driver will then write the logic signal on the databus to each cell by raising a $BL$ to HIGH if a 1 is to be stored, and raising $\overline{BL}$ if a 0 is to be stored.

![[w5n2SRAMMemoryUnit.png]]
The memory unit stores $2^k$ words of $n$ bits (resulting in $2^k\cdot n$ bits total stored) therefore requiring $k$ address bits. When $R/\overline{W}$ = 0 then the data bus is written to memory, when = 1 the data is transferred from memory to the data bus.

![[w5n2SRAMArrayColumn.png]]
An SRAM memory array is usually storing enough values that it is infeasible to store each word on its own line. This is as, for example, an array with 1 million ($2^{20}$) 8 bit ($2^3$) words would have an aspect ratio of $2^{20}/2^3\approx130,000$. Therefore, a column decoder is used, which is effectively a multiplexer that has on one side the bit lines of every word in a word line and on the other a data bus with a width of 1 word. This means that every read/write op actually opens all access transistors across all words in a line. This allows for much larger SRAM arrays at the cost of increased power consumption (as many more bit lines must be raised to HIGH on each R/W).

