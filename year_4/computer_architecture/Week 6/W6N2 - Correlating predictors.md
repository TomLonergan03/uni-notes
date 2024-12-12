[[W6N1 - Branch prediction continued#1-bit prediction|1-]] and [[W6N1 - Branch prediction continued#2-bit branch prediction|2-bit]] branch predictors use the history of the current branch (**local correlation**), but branch results may also be correlated with the past outcome of other branches (**global correlation**).

E.g. in this snippet the for loop branch result follows the same pattern each time the loop is run:
```c
for (int i = 0; i < 4; i++) {
	x = a[i];
	if (x == 0)
		continue;
	gtz++;
}
```
Branch outcomes: 1, 1, 1, 0,    1, 1, 1, 0,    1, 1, 1, 0,    ...

In this snippet the result of different branches are correlated:
```c
if (a == 2)
	a = 0;
if (b == 2)
	b = 0;
if (a != b) {
	...
}
```
If both of the first two branches are taken, then the last branch definitely isn't taken.

And similarly in this snippet:
```c
void reverse_str(char *s) {
	if (s1 == NULL)
		return;
}

char s1 = "Bob";
...
if (s1 != NULL)
	reverse_str(s1);
```
# Global two-level predictors
Here, we have many predictors, often in the thousands. The global history of branch results is stored in a **global history register (GHR)**. Each predictor is a 2-bit saturating counter, and the predictor is selected using the PC to select the row of the **pattern history table (PHT)**, while the history bits select the predictor from that row. This means that the predictor used is based on the results seen in the previous times there has been the same pattern of branches leading up to the current branch.

In practice this isn't possible, as the table cannot be $\text{total addressable instructions}\cdot2^\text{GHR bits}\cdot2$ bits total. Instead, some of the bits of the PC is used to index, or by using a hash function to combine the PC and GHR into one value to index a PHT. One common hash function is Gselect, with `PHT index = {PC[n:m], GHR}`, another is Gshare, with `PHT index = PC[n:2] ^ GHR`.

![[w6n2correlatingPredictorPerformance.png]]
Here, we see that a 2-level correlating predictor can improve prediction accuracy by 4% or more relative to a 2-bit branch predictor, with the same total number of predictors.

This is a significant performance improvement, especially on modern processors with deep pipelines.

If branches resolve in stage 10, 20% of instructions are branches, and there is no penalty for correctly predicted branches, then:
$$
\begin{aligned}
\text{2-bit predictor CPI }&= 0.8+0.2\cdot(10\cdot0.08+1\cdot0.92)=1.114\\
\text{2-level predictor CPI }&= 0.8+0.2\cdot(10\cdot0.04+1\cdot0.96)= 1.072
\end{aligned}
$$
# Branch target buffers
The **branch target buffer (BTB)** stores the target addresses of branch decisions, as we need to know what the branch target is to be able to resolve a branch early.

![[w6n2branchPredictionLogic.png]]
This is an example of branch prediction logic, using a global predictor for predicting if a branch is taken, and the PC to select the target from the BTB.

The first time a branch is seen, the branch will be mispredicted as it is not yet in the BTB (even if the PHT correctly predicts a branch). When the branch is resolved, this misprediction will be recorded and the branch target will be entered into the BTB.
# Tournament predictors
Multiple branches may index into the same predictor in the PHT, which can interfere with other predictions on other branches.
The **tournament predictor** expects that most branches are biased (e.g. taken 99% of the time), and these branches can be predictor with a simple predictor such as a local 2-level with a short history to reduce interference, and hard branches can be predicted with a global 2-level predictor. A meta-predictor is then used to select which of these predictions is used, using a PHT of 2-bit saturating counters.
![[w6n2tournamentPredictor.png]]

# Branch prediction overall
![[w6n2branchPredictorsOverall.png]]