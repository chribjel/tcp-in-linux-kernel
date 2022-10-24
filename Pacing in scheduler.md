TCP can now control packet departure time explicitly after *Earliest Departure Time* was introduced in the kernel. This allows TCP to attach a timestamp to an *skb* which can tell the scheduler when to send the packet. Only a few qdiscs support this, including *fq/pacing* and *ETF*.

For our thesis, I imagine all pacing will be handled in the TCP stack, as we will handle the timers ourselves as we need fine grained control. When we send packet from TCP, we want it to be sent immediatly. This can be handled with a simple classless FIFO queue in the scheduler. 

TCP: switch to Early Departure Time model:
https://lwn.net/Articles/766564/


skb_mstamp_ns