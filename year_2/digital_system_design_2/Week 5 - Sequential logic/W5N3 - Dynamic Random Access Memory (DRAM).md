- **Dynamic** - must be frequently refreshed to maintain memory contents
	- This is due to leakage of charge from the capacitor
	- Refresh mechanism means DRAM is slower than [[W5N2 - Static Random Access Memory (SRAM)|SRAM]]
- **Random access** - information can be accessed in any order
- **Memory** - stores bits for later use
- **Volatile** - memory content is lost when powered off

![[w5n3DRAMCell.png]]
Stores a bit by charging/discharging $C$.

As DRAM only uses 1 capacitor and 1 transistor, a DRAM cell is much smaller than an SRAM cell. This means that DRAM is both cheaper and denser storage

An array is laid out similar to an [[W5N2 - Static Random Access Memory (SRAM)#SRAM Array|SRAM array]], though R/W ops occur differently. Writing occurs by writing the desired HIGH/LOW to $BL$ in order to read/write to the cell.
Reading is done by pre-charging $BL$ to $V_{PRE}=V_{dd}/2$, then setting $WL$ to HIGH. Therefore, charge redistribution occurs between $C$ and the parasitic capacitance of $BL$, referred to as $C_{BL}$. The change in voltage is described by:
$$\Delta V=V_{BL}-V_{PRE}=(V_{BL}-V_{PRE})\frac{C}{C+C_{BL}}$$
The voltage change on $BL$ is sensed, as the voltage will fall if $C$ is LOW and rise if $C$ is HIGH. After the read, the value is written back to the cell to preserve the stored value.

# DRAM control signals
![[w5n3DRAMMemoryUnit.png]]
The row address is strobed onto the address bus on the falling edge of $\overline{RAS}$, then the column address is strobed onto the address bus on the falling edge of $\overline{CAS}$.
If $\overline{WE}$ is HIGH then the cell reads the address, if it is LOW then the cell writes the address.

![[w5n3DRAMWriteOp.png]]
An example write operation.