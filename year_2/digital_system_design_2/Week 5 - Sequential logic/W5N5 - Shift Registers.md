# Serial input, parallel output (SIPO) shift registers

By connecting 2 or more [[W5N1 - D flip-flop|D flip-flops]] in series, we have a register that will shift the values in it 1 bit per cycle to the right or left.
![[w5n5SIPOShiftRegister.png]]

This means that we can input data on $D_{in}$ serially (1 bit at a time), and read the output at any point.

## Usecase - switch debouncing
A use for a SIPO register is to "debounce" a switch, as when a mechanical switch is closed it bounces many times before properly closing, which could lead to an incorrectly oscillating logical output.
![[w5n5ShiftRegisterDebouncing.png]]
This circuit means that if the switch has been closed at any point in the last 8 milliseconds, then $OUT$ will be HIGH, otherwise it will be LOW.

# Parallel input, serial output (PISO) shift register
By cascading [[W5N6 - Multiplexer|multiplexers]] and D flip-flops, we can produce a shift register that will read in a number of bits in parallel, then output them one at a time.
![[w5n5PISOShiftRegister-1.png]]
(if we input 0101)

![[w5n5PISOShiftRegister.png]]
When $Load/\overline{shift}$ is HIGH, then the values $B_0,...,B_n$ are stored in the register.
When $Load/\overline{shift}$ is LOW, then the values stored are sequentially output to $D_{out}$, one per clock cycle.

## Usecase - Serialising data
A use for a PISO register is to convert parallel data, such as stored in a computer, to a serial format for transmission between computers over radio or through cables.
Data transmission is a common source of error, so we can use a [[W5N7 - Parity bits|parity bit]] to check if data was corrupted during transmission.

We then add:
![[w5n7ParityLogicCircuit.png]]
to our PISO register to get:
![[w5n5PISOParity.png]]

## Serial receiver with parity check
![[w5n5SIPOParityCheck.png]]
Now, if $parity\text{ }check$ is HIGH we know that there was an error in transmission that resulted in 1 bit being flipped.