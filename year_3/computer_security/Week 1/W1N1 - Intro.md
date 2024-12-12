# What is security?
A secure system generally achieves the following (and some other things):
## Confidentiality
- Information is only accessed by authorised users
## Integrity
- Data is untampered and uncorrupted
## Availability
- Both the data and the system that provides access to it are there when you need them

# What is trust?
Generally, we trust when we have:
## Assurance
- The means to know that a system is secure

## Reliability/resilience
- Operates intact in the face of natural disasters and man-made attacks

## Accountability
- The means to verify that the system is operating as designed

# What is privacy
Privacy concerns individuals and how their data, behaviour, and interactions are recorded, utilised, and distributed.

A person gets to control information about themselves:
- Who gets to see it
- Who gets to use it
- What they can use it for
- Who they can give it to

# Who are the adversaries
- Nature
	- Natural disasters can have significant impact on systems
- Script kiddies
	- Amateur hackers deploying public domain attacks for amusement/to cause annoyance/for revenge
- Crackers/hackers
	- Skilled hackers creating novel exploits for money/bragging rights
- Organised crime
	- Groups actively trying to make money
	- Often have serious backing/resources
- Governments
	- Either domestic or international, trying to exert pressure on other states/companies/individuals to further national interests
	- Vast resources and nearly unlimited money
- Terrorists
	- Individuals or groups trying to create change through mass fear/damage

When designing a system we must decide who we are worried about.
e.g. a multiplayer game would be worried about script kiddies and hackers, but might accept it is not capable of resisting a serious government backed attack.

# Threat modelling
When designing a system we must determine:
- Who is the adversary
- What can't we prevent them from doing
- What do they want
	- What do we want to stop them from doing
- The set of threats we want to protect against these adversaries

# Terminology
- **Assets:** things we want to protect e.g. hardware, software, information
- **Vulnerabilities:** weaknesses in a system that may be exploited
- **Threats:** what are we worried about. Generally loss or damage to the system/its users/operators
	- 6 major categories of threats:
		- **Interception:** a message is sent, but is secretly redirected to somewhere else
		- **Interruption:** a message is prevented from reaching its destination
		- **Modification:** a message is altered between being sent and being received
		- **Fabrication:** a fake message is sent that looks real to the receiver
		- **Repudiation:** a message is sent, and later claimed to have not been sent (with no proof of it having been sent)
		- **Epistemic:** a system/person is receiving false information to induce it to make incorrect decisions
- **Attack:** an action which exploits a vulnerability to carry out a threat
- **Controls:** a mitigation or removal of a vulnerability
	- No system is perfect, so vulnerabilities will need to be controlled as they are discovered during a system's lifetime

# Defence methods
- **Prevent** an attack from occurring
- **Deter** people from attempting an attack
- **Deflect** them to attack something else
- **Detect** an attack before/while its happening
- **Recover** after an attack with minimal loss

Best practice employs some or all of these patterns

## Defence tools
- **Cryptography:** encrypt data, both in storage and transit, authenticate users and transactions, ensure integrity of data against unauthorised modification
- **Software control:** access controls, permissions, software firewalls, passwords, sandboxes, virus scanners, version control systems
- **Hardware controls**: physical isolation of systems or access
	- Fingerprint readers, smart tokens, firewalls, intrusion detection systems
- **Physical control:** limiting access to the hardware
	- Locks, guards, offsite backups
- **Policies and procedures:** ensuring that a single malicious/incompetent individual is not capable of causing serious harm by requiring reviews/approvals from others or by enforcing specific workflows
	- Disallow personal devices in a sensitive area, password rules, security training, password managers