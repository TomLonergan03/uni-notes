While a [[W2N4 - Firewalls|firewall]] is preventative, an **IDS** detects potential incidents while they occur.

Most security incidents occur when a user lets something malicious into the network, or by an inside threat. These cannot be prevented or anticipated with any certainty. An IDS identifies that something bad is happening, hopefully quickly enough that it can be addressed and mitigated.

# Rule based intrusion detection
- A ruleset identifies the types of actions that match known intrusion attacks
- This requires an admin to anticipate attack patterns in advance, although an attacker may test an opening using a common signature before they try something novel so some novel attacks will come with warning
- It is, however, impossible to detect a new form of attack
- High accuracy, low false positives
# Statistical intrusion detection
- Dynamically builds a statistical model of "normal" behaviour and flags anything that does not match
- This means the admin doesn't need to anticipate potential attacks
- The system takes time to "warm up" to new behaviours, and may flag new intended behaviours for a period after they are introduced
- Higher false positives, lower accuracy

## Base rate fallacy
In a system with a low rate of malicious events, the false positive rate is far more impactful on the number of false detections.
E.g.
If an IDS is 99% accurate, with a 1% chance of false positives or negatives, then:
- If it generates 1,000,100 log entries
- 100 of those correspond to actual malicious events
- Of the 100 malicious events, 99 will be detected, and there will be **1 false negative**
- Of the 1,000,000 benign events, 10,000 will be mistakenly flagged as malicious, so we will have **10,000 false positives**
- Thus, there will be 10,099 alarms, of which 10,000 are false alarms. Therefore, roughly **99% of raised alarms are false alarms**

# Number of alarms
The number of alarms raised by an IDS is a big problem, as if most alarms are false positives people will tend to ignore them over time, even if there is a real attack. It also makes it hard to identify which alarms need investigated once it is determined that an attack is occurring.
Additionally, some cyberattack insurance policies state that if you know about an attack and do nothing then they will not cover the attack.