Computers are used in many safety-critical applications, e.g. automotive, aeronautical, and medical, and these are covered by various standards such as ISO 26262 and ASIL. This means that we want to detect and correct errors at the hardware level to prevent systems from malfunctioning.
# Failures
**Soft errors** have a specific operation fail, which may corrupt data, but the component will continue functioning correctly on subsequent operations, e.g. writing a value to RAM with a flipped bit as future writes to the same address will succeed.

**Hard errors** have a component fail and stay in error, e.g. a RAM chip starts producing random values on read or a wire disconnects.

Failures can occur in many places, here ordered roughly from most common to least common:
- Main memory, especially large DRAMs
- Large on-chip caches
- Register files
- High speed busses, even those on-chip

In extremely safety-critical applications, all of these are checked for errors, and there may even be duplicate processors or memories which both run exactly the same instructions on the same data, and if they ever disagree it's detected as a fault and the device enters a failure recovery mode (dual-core lock-step).

Failures can be caused by:
- Out-of-tolerance devices leading to excessive cross-talk (current on one wire may induce current in other nearby wires), or burn-out
- External factors such as temperature, humidity, ionising radiation
## Single event upsets
**Single event upsets (SEUs)** are single bit-flips caused by ionising radiation or electrical noise. There are two primary sources of radiation:
- Atmospheric neutrons, from the impact of cosmic rays on the Earth's atmosphere can have energies up to 10000 MeV. These are most common at high altitudes, but do still occur at sea level
- Alpha particles from radioactive impurities in chip packages, with $^{238}\text{U}$ and $^{232}\text{Th}$ yielding energies of 4-9 MeV

Protective packages reduce $\alpha\text{SEUs}$ by 98%, but cosmic particles remain, as even 1m of concrete reduces neutron flux by only 30%. This means that high integrity systems must be able to tolerate SEUs caused by sea-level neutrons.
# Error correction codes (ECCs)
To detect errors, we calculate an extra value based on the data we are writing, and if a bit gets flipped the check value will not match the value calculated from the data. If we carefully design the check bits, we can also determine where the error occurred, and correct it.
## Detection vs correction
**Detection** flags an error when it occurs, while **correction** detects an error and fixes it. In both cases, capability is measured in the number of errors tolerated, e.g. 1-bit detection means that if two bits are flipped then there is no guarantee that the error is detected. Larger detection and correction capabilities require more bits in the ECC, and correction requires more bits than detection.

The **failures in time (FIT) rate** is the number of failures occurring per billion device hours of operation.
## Parity
One **parity** bit per protected data unit will provide 1-bit error detection. The parity bit is set such that the number of 1s in the data is even.
![[w10n3parity.png]]If the parity bit itself is flipped, this is also detected and counts as one error.
## SECDED
**Single error correction, double error detection (SECDED)** is an ECC that uses parity bits in powers of two positions ($1,2,4,8,...$) with the parity bit $b$ checking bit $m$ if $m$ has bit $b$ set to 1 in its binary representation, e.g. position 1 checks bits $1, 3, 5, 7, ...$, position 2 checks bits $2,3,6,7,...$, position 3 checks $4,5,6,7,12,...$.

To protect 8 data bits we need 4 parity bits:
![[w10n3secded.png]]
We create the parity bits and insert them in the correct locations, then on receipt of the data we recreate the expected parity bits $\{e_4,e_3,e_2,e_1\}$ using the same computation, and form a syndrome $S=\{e_4,e_3,e_2,e_1\}\wedge\{p_4,p_3,p_2,p_1\}$. If $S\ne0$, then $S$ is the position of the flipped bit.
![[w10n3secdedCorrection.png]]

In verilog, we can calculate the check bits as follows:
```verilog
module secded_check_bits_16bit (
	input [15:0] D, // data to write to memory
	output reg [21:0] W // data + check bits + parity
);
	
reg P0, P1, P2, P3, P4; // check bits

always @* begin
	// Compute check bits P from selected input data bits
	//
	P0 = ^(D & 16’b1010110101011011);
	P1 = ^(D & 16’b0011011001101101);
	P2 = ^(D & 16’b1100011110001110);
	P3 = ^(D & 16’b0000011111110000);
	P4 = ^(D & 16’b1111100000000000);
	// Interleave check bits P with D to make write data W.
	// Each Pi is inserted into bit W[2^i], and the
	// 16 data bits are inserted in the gaps between the
	// Pi bits, starting from the least-significant end
	// of W and D.
	//         21-------16----------8---------4-------2--1
	W[21:1] = {D[15:11],P4,D[10:4],P3,D[3:1],P2,D[0],P1,P0};
	// Compute bit W[0] as the parity over the entire
	// group of interleaved data and check bits, for
	// double-error detection.
	//
	W[0] = ^W[21:1];
end
endmodule
```

and correct any errors:
```verilog
module secded_correction_16bit (
	input [21:1] D, // data read from memory
	output reg [15:0] R // post-correction data
);

reg [4:0] E, P, S; // check bits and syndrome
reg [21:0] C; // corrected memory word

always @* begin
	// Re-compute check bits E using read data from memory
	//
	E[0] = ^(D & 21’b101010101010101010101);
	E[1] = ^(D & 21’b001100110011001100110);
	E[2] = ^(D & 21’b110000111100001111000);
	E[3] = ^(D & 21’b000000111111110000000);
	E[4] = ^(D & 21’b111111000000000000000);
	
	// Extract the check bits P from the read data
	//
	P = {D[16],D[8],D[4],D[2],D[1]};
	// Compute the syndrome S
	//
	S = E ^ P;
	// Toggle bit S to correct errors in bits [21:1],
	// and extract relevant data bits
	//
	C = (22’d1 << S) ^ {D, 1’b0};
	R = {C[21:17], C[15:9], C[7:5], C[3]};
end
endmodule
```
## Application of SECDED
SECDED is used for server DRAM. For safety-critical systems, it is used on all external memory data values, and all address pathways from CPU to memory, both internal and external.

A study of Google servers showed FIT rates of 25,000-70,000 per Mbit of DRAM, with >8% of DIMMs affected by errors each year. Failures were dominated by hard errors, but 65-80% of un-correctable DRAM errors were preceded by a correctable error in the same month, and age was an important factor for failure rates.