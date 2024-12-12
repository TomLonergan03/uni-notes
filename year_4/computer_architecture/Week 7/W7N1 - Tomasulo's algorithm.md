[[W6N4 - Scoreboarding|Scoreboarding]] still results in stalls, allows out-of-order writes to result registers, and has no [[W4N1 - Pipeline hazards#Data forwarding|data forwarding]]. **Tomasulo's algorithm** avoids WAW and WAR using dynamic register renaming, and supports forwarding, while still giving the benefits of out-of-order execution.
# Key ideas
Instead of centralising everything in a scoreboard, controls and buffers are distributed with functional units, called **reservation stations**. Register names are replaced by pointers to reservation stations (register renaming), and a common data bus broadcasts results to all functional units. 
## Reservation stations
A **reservation station (RS)** contains the instruction, operand values (when they are available), and the RS numbers of the instructions providing the operand values. An instruction is assigned to an RS on issue, and any RS that is able to execute can be run any time there is an available functional unit. As there can be many more reservation stations than registers (and reservation stations are a microarchitectural detail) it can be very unlikely that an instruction ever has to block for a reservation station to become available.
![[w7n1reservationStation.png]]
## Step by step
1. **Issue**
	- Get next instruction from the fetch queue and issue it to a reservation station if there is a free RS
	- Read operands from register file if they are available, otherwise link the RS to the producing instructions' RS
2. **Execute**
	- Monitor results on the **common data bus (CDB)**, if the CDB has a result needed by a reservation station it is copied by that RS
	- Execute instruction when both operands are ready in the RS
	- Loads/stores are managed in a separate queue that preserves program order by tracking effective addresses
	- All preceding branches must resolve before an instruction can execute
3. **Write result**
	- Broadcast result on CDB, and write it to the regfile if it is the most recent producer for that register
## Loads and stores
Loads and stores are placed in a dedicated set of buffers in program order, and reordering those buffers is not allowed. This is to prevent dependency violations through memory.
## Common data bus (CDB)
The CDB transports both the data and the RS that is the source of that data. The RS information is used so that dependent instructions can match. Each register in the register file is tagged with the RS# of the most recent instruction that will write to it, and only CDB results with that RS will write to the regfile.
## Book-keeping
Reservation stations have:
- **Busy**: whether a station is in use
- **Op**: the operation to be performed
- **Q$_i$, Q$_j$**: reservation stations of source operands, or null if no dependency
- **V$_j$, V$_k$**: values of source operands, captured at issue if there is no dependency or later from the CDB

The register file has:
- **Q$_i$**: indicates which RS will write each register, if one exists, otherwise null
# Summary
## Advantages
- Register renaming means we don't need to stall to prevent WAR and RAW
- Allows forwarding
## Limitations
- Branches stall execution until after the branch is resolved, so no speculation beyond branches
- Implementing Tomasulo's beyond just floating-point operations introduces the risk of imprecise exceptions, which complicates exception recovery
