As the [[W3N3 - RISC-V pipelined instruction execution|pipeline]] gets deeper, the [[W4N1 - Pipeline hazards|cost of taking a branch]] increases, increasing [[W1N1 - Principles of computer architecture#Factors that affect CPU performance|CPI]]. One way to reduce this impact is through **branch prediction**: attempting to predict whether a branch occurs and speculatively executing the instructions after that branch.
# In the RISC-V pipe
![[w4n2branchPrediction.png]]
The **branch prediction unit (BPU)** predicts whether a branch will occur, and that prediction defines the next fetch PC address. First, the BPU predicts if a PC value is going to be a branch before the instruction is even decoded. This prediction travels through the pipeline along with the instruction, and after the instruction is decoded it confirms that that instruction is a branch, then after EXE the branch test result is known, and the branch is either taken or not taken. If the prediction was incorrect at IF, then everything in the pipeline before MEM is wrong and gets flushed. If the prediction is correct then nothing happens. In either case, the prediction result is fed back to the BPU, which uses it to update its internal state to improve future prediction accuracy.
# Impact of branch prediction on CPI
If the cost of a branch is 1 cycle when predicted correctly and 5 cycles if mis-predicted, then on average $CPI_\text{branch}=(P(\text{incorrect})\cdot5)+(P(\text{correct})\cdot1)$
e.g. if $P(\text{correct})=0.95$ then $CPI_\text{branch}=(0.05\cdot5)+(0.95\cdot1)=1.2$
In this case, if 20% of instructions are branches and $CPI_\text{non-branch}=1$ then $CPI=0.8\cdot1+0.2\cdot CPI_\text{branch}=1.04$
This predictor therefore improves CPI from 1.4 to 1.04, showing the power of branch prediction.

As the pipeline gets deeper, the branch predictor must get more accurate, as the cost of a mis-prediction increases.