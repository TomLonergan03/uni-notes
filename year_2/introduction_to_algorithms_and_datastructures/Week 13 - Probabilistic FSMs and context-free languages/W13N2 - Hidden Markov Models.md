An Hidden Markov Model (HMM) is a transducer [[W13N1 - Probabilistic Finite State Machines|probabilistic FSM]], which means that it no longer tests input strings, but generates output strings based off of what states it enters. Each state will generate a character from $\Sigma$ according to some probability distribution for $\Sigma$ in that state.

An example HMM which models weather sequences:
![[w13n1WeatherHMM.png]]
The outputs could represent the weather, with one output per day. The HMM doesn't give us the start state distribution, $\pi$, but we will assume all states are equally likely.
For example, if we start in $q_1$, there is a 0.2 chance of outputting clouds, a 0.6 chance of outputting rain, and a 0.2 chance of outputting hail. Then on the next day, there is a 0.3 chance of remaining in $q_1$, a 0.3 chance of transitioning to $q_3$, and a 0.4 chance of transitioning to $0.4$.