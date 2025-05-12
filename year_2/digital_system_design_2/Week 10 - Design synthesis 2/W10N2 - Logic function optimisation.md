[[W1N2 - Boolean algebra|Boolean algebraic transforms]], term minimisation (through Karnaugh maps), and other methods may be applied to truth tables, either individually or jointly (called common term sharing) in order to meet performance targets.
At this stage, any "don't care" conditions (\*s) can be exploited. This includes partial decoding of [[W8N3 - State code design#Impact of density on state codes|sparse state encodings]] where the evaluation of a few key bits is enough to recognise state, implicitly making the other bits "don't care".

For some [[W8N1 - Fabrics|fabrics]] or applications, specific optimisations may be weak or useless, so use is dependent on design and commercial considerations.

# Mapping logic to physical gate structure
For an [[W8N2 - FPGAs#The logic cell|FPGA logic cell]], fan-in is the only limit while the Boolean function complexity is otherwise irrelevant due to using LUTs.
For an [[W8N1 - Fabrics#Custom gate ASIC|ASIC]], fan-in is far more flexible, while Boolean function complexity is the primary decider of the logic area required.

# Routing
Finding a perfect routing is impossible [(at least as far as we can tell)](https://en.wikipedia.org/wiki/Travelling_salesman_problem). Therefore, the standard tactic for generating placings and routings is through random chance, by using synthesis tools which generate millions of possible layouts (within bounds and guided by heuristics) and attempts to join them up. Each solution is then measured for quality according to some scoring system, and repeated until a result that meets a set standard is found.