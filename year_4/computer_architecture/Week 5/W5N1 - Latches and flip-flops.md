# Set-reset latch (SR latch)
![[w5n1srLatch.png]]
An SR latch can be "set", where it outputs a high signal, or "reset", where it outputs a low signal. An SR latch also provides complementary outputs.
# SR latch with enable
![[w5n1srLatchEnable.png]]
By adding an enable signal which is NAND with both $S$ and $R$ signals, we can make an SR latch that only changes state when the enable signal is enabled.
# D-latch
![[year_4/computer_architecture/images/w5n1dLatch.png]]
When $En=1$, the latch is "open" and $Q$ follows $D$. When $En=0$, the latch is "closed" and $Q$ retains its current value.
# D-type positive-edge-triggered flip-flop
![[w5n1posedgeFlipFlop.png]]
A positive edge triggered flip-flop stores the value on $D$ at the rising edge of each clock cycle.
