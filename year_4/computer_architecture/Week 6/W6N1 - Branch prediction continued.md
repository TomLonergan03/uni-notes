There are two main approaches to [[W4N2 - Branch prediction|branch prediction]]:
- **Static prediction** where branch predictions are made for each branch before program execution
- **Dynamic prediction** learns from branch results while the program runs, and uses that information to predict future branch results
# Static prediction
One approach for static prediction is that the compiler predicts the result of each branch, and encodes that into the branch instructions it emits, e.g. a bit in the instruction where 1 = branch likely, 0 = branch unlikely. Another approach is to profile specific programs, or a general selection, and base predictions on that. Some predictors assume **"backwards taken, forward not taken" (BTFN)**, if e.g. 90% of backward branches are taken (e.g. in loops), while <50% of forward branches are taken. BTFN has been used in older processors, but is no longer commonly used.
# Dynamic prediction
Dynamic branch prediction predicts the future branch results of individual branches (identified by PC) based on the past results of that specific branch, and can predict both the outcome of the branch and the target address of the instruction to branch to. After a branch occurs, the prediction is compared to the actual result, and update the predictor and if necessary flush the pipeline.
## Dynamic prediction algorithms
### 1-bit prediction
The simplest branch predictor stores each branch PC, and 1 bit indicating if it was taken (1) or not taken (0) last time it was seen.
![[w5n11bitPrediction.png]]
This tends to be unstable, and in cases where the outcome of a branch tends to alternate results in all predictions being incorrect, which can be worse than simply predicting one of taken or not taken every time.
### 2-bit branch prediction
Adding hysteresis protection reduces the impact of one-off events on predicted outcomes. This can be done with a 2-bit saturating counter, where 00 and 01 result in branches being predicted as not taken, and 10 and 11 result in branches being predicted as taken.

| Branch PC | Outcome |
| :-------: | :-----: |
|  0x135c8  |   10    |
|  0x147e0  |   11    |
|    ...    |   ...   |
We can represent this predictor as a state machine:
![[w6n12bitPredictorStm.png]]
This can be extended to an $N$-bit predictor, where if the counter $>(2^n-1)/2$ a branch is predicted as being taken. Generally, $N$-bit predictors are not much better than 2-bit predictors, as the extra space they use can be better used tracking extra state for each branch.