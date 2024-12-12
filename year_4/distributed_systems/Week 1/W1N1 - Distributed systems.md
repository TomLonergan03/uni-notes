A distributed system is a collection of multiple computers or nodes communicating via a network to achieve a task collectively. They appear as a single coherent system to users. Examples include the Internet, most large websites (Google/Spotify etc.), and large scale machine learning systems.

Distributed systems are used:
- For reliability, to allow a system to continue working even when individual computers/components fail
- For performance, to process more data than a single node can handle

Distributed systems are difficult as:
- Communication links can fail
	- A network may become partitioned for a period, then reconnect requiring data coherency to be reestablished
- Individual nodes can crash
	- Faults happen randomly or in coordination
- Security and privacy concerns with data and processing being spread over many locations/jurisdictions

Design goals for distributed systems typically include:
- Supporting sharing of resources: use resources efficiently and cost effectively
- Distribution transparency: hide the distributed nature of processes and resources from the end user
- Openness: components can be easily used or integrated into other systems
- Dependability: be relied upon to operate as expected
- Security: ensure confidentiality and integrity
- Scalability: scale well with more resources and/or users
