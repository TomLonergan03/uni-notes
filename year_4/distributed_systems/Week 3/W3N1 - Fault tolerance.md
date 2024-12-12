# Reliability
**Reliability** $R(T)$ of a component $C$ is the conditional probability the $C$ has been functioning correctly during $[0,t)$ given $C$ was functioning correctly at time $T=0$.
The traditional metrics for reliability are:
- **Mean time to failure (MTTF)**: the average time until a component fails. Ideally this is infinite, but really it'll just be as high as possible
- **Mean time to repair (MTTR)**: the average time needed to repair or replace a component. Ideally this is as close to zero as possible
- **Mean time between failures (MTBF):** MTTF + MTTR
# Availability
**Availability** $A(t)$ of component $C$ is the average fraction of time the $C$ has been functioning correctly during $[0,t)$. 
$$\text{Availability}=\frac{MTTF}{MTBF}$$
Availability is often given as $n$ 9s, e.g:
- Two 9s (99%) allows for 3.65 days/year of downtime
- Three 9s (99.9%) - 8.7 hours/year
- Four 9s (99.99%) - 52.58 minutes/year
- Five 9s (99.999%) - 5.25 minutes/year
# Terminology
- **Failure**: the system as a whole is not working
- **Error**: part of the component that can lead to a failure
- **Fault**: cause of an error
- **Fault tolerance**: system continues to work in the presence of faults
## Fault categories
- **Transient**: the fault occurs once and disappears
- **Intermittent**: occurs many times in a random fashion
- **Permanent**: continues to exist until fixed/repaired/replaced
# Byzantine generals problem
If we have a number of nodes that all must agree on an action, with some of those nodes being malicious (or faulty), we must have a process to agree on what will happen. In general, we need $3f+1$ nodes in total to tolerate $f$ malicious nodes.
# System models
A system model captures our assumptions about how nodes and the network behave.
## Network behaviour
- **Reliable**: messages will be received if it is sent, but they may be out of order
- **Fair-loss**: messages may be lost, duplicated, or reordered
- **Arbitrary**: a malicious adversary may eavesdrop, modify, spoof, or replay messages

A fair-loss network can be made reliable, using something like [[W5N2 - TCP|TCP]], and an arbitrary network can be made fair-loss, using something like [[W4N3 - Digital signatures|signatures]] and [[W4N2 - Public key encryption|encryption]].
## Node behaviour
- **Fail-stop**: nodes crash on failure, and can reliably detect failure
- **Fail-noisy**: nodes crash on failure, but only eventually reliably detect failure
- **Fail-silent**: omission or crash failures, so the client cannot tell what went wrong
- **Fail-safe**: arbitrary failures which do not cause harm
- **Fail-arbitrary**: arbitrary failures which may cause harm

Node behaviour cannot be converted between models, it is set in the design of the system.
## Timing assumptions
- **Synchronous systems**: process execution speeds and message delivery times are bounded, so omission and timing failures can be detected
- **Asynchronous systems**: no assumptions can be made about process execution speed or message delivery times, so crash failures cannot be reliably detected
# Types of failures
- **Crash failure**: system works fine, then halts
- **Omission failure**: system fails to respond to incoming requests (receive omission), or fails to send a response (send omission)
- **Timing failure**: response lies outside a specified time interval
- **Response failure**: response is incorrect, either **value failure**, where the value of the response is wrong, or **state-transition failure**, where a deviation occurs from the correct control flow
- **Arbitrary failure**: may produce arbitrary responses at arbitrary times
# Failure detection
A perfect failure detection labels node as faulty if and only if it crashed. For a crash-stop system, the typical implementation will declare a node crashed if it doesn't respond to a message within a certain timeout. Reliable failure detection is practically impossible unless it's a synchronous system with crash-stop behaviour.
# RPC fault tolerance
[[W2N1 - Communication#Remote procedure calls (RPCs)|RPC]] has 5 different fault scenarios:
- The client cannot locate the server
	- Reason: servers are down, or a mismatch in client-server stub versions
	- Solution: raise an exception or return error value in the client
- The server crashes after receiving a request
	- Reason: software or hardware failure, the client can't tell whether the server crashed before or after executing the request
	- Two solutions: the server can guarantee either that an operation will be carried at least once, no matter what, or the server can guarantee it will carry out an operation at most once
		- After the server recovers, clients can:
			- Always reissue the request
			- Never reissue the request
			- Reissue the request if no acknowledgement is received from the server
		- Which of these is chosen depends on the guarantees the server provides
- The request message is lost
	- Reason: network failure
	- Solution: the client retransmits if no acknowledgement is received within a timeout 
- The reply message is lost
	- Reason: network failure or the server is over capacity and gets stuck for a while
	- Solution: same as lost request message, but the server must be careful. Idempotent operations like reads are safe to repeat, but non-idempotent operations like append/create/delete must not be repeated so the server must filter out duplicate requests
- The client crashes after sending a request
	- Reason: software/hardware failures
	- Solution: there may be an orphaned response for the request that might need to expire, best thing is to not do anything and attempt to restore the client to the state it was in before the crash