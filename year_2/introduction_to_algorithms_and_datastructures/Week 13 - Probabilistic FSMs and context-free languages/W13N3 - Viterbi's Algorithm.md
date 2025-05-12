Given an [[W13N2 - Hidden Markov Models|HMM]] output and that HMM, what is the most likely path through the HMM?

Using this weather modelling HMM:
![[w13n1WeatherHMM.png]]
What is the most likely path to have been taken to produce this output?
![[w13n3WeatherHMMOutput.png]]
We could have $q_1$, $q_1$, $q_1$, $q_2$, $q_3$ or $q_3$, $q_3$, $q_3$, $q_2$, $q_3$ or many other paths to produce this output.

# Computing the maximum likelihood path
The output of the HMM, $s=s_1...s_n$
An optimum path must end at some state $q$.
The path must arrive there through an incoming transition from $q*\rightarrow q$.
The probability of this transition then emission is $p_{q*\rightarrow q}\cdot b_{q,s_n}$, where $p_{q*\rightarrow q}$ is the probability of a transition from $q*$ to $q$, and $b_{q,s_n}$ is the probability that $s_n$ is emitted in state $q$.
For every $q*$ except the initial state, there is a maximum likelihood path that ends at that $q*$, and in the initial state, the likelihood is $\pi_q\cdot b_{q,s_1}$, or the chance of starting in that state times the probability of emitting the initial starting character.

## Recurrence
$$
mlq(i,q) = \left\{\begin{matrix}
	\pi_q\cdot b_{q,s_1} & \text{if }i=1\\
	max_{q*\in Q}(mlp(i - 1, q*)\cdot p_{q*,q}\cdot b_{q,s_i}) & \text{if }i>1\\
\end{matrix}
\right.
$$
$mlp(i,q)$ is the most likely path to generate $s_1,...s_i$ which ends at $q$.
It isn't required to check if $q*\rightarrow q$ is a valid transition if we define all transitions with any invalid ones having a probability of 0.

# Implementation
If we add an array `prev` such that `prev[i, q]` is the state $q*$ that optimises the `mlp[i,q]` value then we can reconstruct the optimum path.

```
Viterbi(hmm,s):
	for state in hmm.states:
		mlp[1, state] = hmm.start_dist[q] * hmm.emission_prob[q, s[1]]
		prev[1, state] = "-"
	for i = 2 to n:
		for state in hmm.states:
			mlp[i, state] = 0
			prev[i, state] = "-"
			for previous_state in hmm.states:
				transition = hmm.transition_prob[previous_state, state] * hmm.emission_prob[q, s[1]]
				if (transition * mlp[i - 1, previous_state]) > mlp[i, state]
				mlp[i, state] = transition * mlp[i - 1, previous_state]
				prev[i, state] = previous_state
	max = 0
	for state in hmm.states:
		if mlp[n, state] > max:
			max = mlp[n, state]
			maxstate = state
	return max, maxstate
```

# Asymptotics
Runs in $\Theta(n\cdot|Q|^2)$ time, as the loop to n contains a double nested loop to $|Q|$. Uses $\Theta(n\cdot|Q|)$ space for `mlp`.