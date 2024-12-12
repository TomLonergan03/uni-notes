[[W6N4 - Scoreboarding|Scoreboarding]] and [[W7N1 - Tomasulo's algorithm|Tomasulo's algorithm]] both write register results out-of-order, which makes it difficult to restart after an exception, such as a page fault. This means that state updates must occur in program order, but we don't want to stall instructions on write-back. This is enforced using a **reorder buffer (ROB)**, into which results are placed as soon as they're ready, and from which results are then committed into the regfile in program-order. The ROB acts as a speculative register file.
![[w7n2rob.png]]
# Operating principles
The ROB is a sliding window of all in-flight instructions. The ROB size defines the maximum number of outstanding instructions, with each instruction having a ROB entry allocated at point of issue.

Source registers must be read from the ROB if there are pending writes, and the ROB may contain several writes to the same register, in which case the most recent write must be used.
# Actions
- **Issue**: stall if `is_full(ROB)` otherwise allocate next ROB entry to issued instruction
  ```
	inst.R <- ROB.head; 
	ROB.head <- (ROB.head + 1) % ROB.size // allocate with wrap-around
	ROB[R].Ri <- inst.Ri; // note result register num
	ROB[R].ready <- false;
    ```
- **Write**: place `inst.result` in ROB entry `inst.R` and set status to 'ready'
  ```
  ROB[inst.R].value <- inst.result;
  ROB[inst.R].ready <- true // ROB gets ready result
	```
- **Commit**: continuously unlink ready results from head of ROB
  ```
	while (ROB[ROB.tail].ready) { // commit whenever ROB is ready
		RegFile[ROB[ROB.tail].Ri] <- ROB[ROB.tail].value // write result to RegFile
		ROB.tail <- (ROB.tail + 1) % ROB.size // deallocate with wrap-around
	}
	```
# Speculation with ROB
Using a ROB allows for speculative execution with [[W6N1 - Branch prediction continued|branch prediction]], with branch outcome results being delivered to the ROB like other instructions. This means that if a mispredicted branch is committed, the entire ROB is flushed, and all pointers into the ROB from the register result status table are invalidated, which restores the regfile as the most up to date status for each register.
# Going beyond Tomasulo's Algorithm + ROB
Reservation stations and ROB are expensive hardware structures, and it is possible to do register renaming without reservation stations by mapping logical registers to a larger  physical register file using a renaming table, and tracking which physical registers are speculative or committed.