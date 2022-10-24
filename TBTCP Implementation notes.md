Notes regarding the implementation of TBTCP in the Linux kernel (5.19.3).

## Implementation scope
As of about 2018, TCP modules may explicitly timestamp a packet to determine when it will be sent out on the wire. Prior to this, pacing was handled by the network scheduler where TCP would set a pacing rate. The scheduler (ex. *FQ/Pacing*) would then calculate send times from rate. TCP is ACK clocked, meaning sending of packets/window increases will only be triggered upon the receival of ACKs (or RTOs). 

TBTCP requires very fine grained control over when packets are to be sent. Three timers are used in the algorithm; one for regular transmission, RTO and one to decide when to enter Fast Recovery. The minimum of these timers is the event that will fire first. Several of these events require explicit notice to send a packet to the network. 

This is not possible by creating a TCP module. Rather, we will have to modify the TCP stack to allow modules to have this explicit control over packet sending. The module must also be capable of arming timers and fire off callbacks when these expire.

## Goals
- Eliminate some ACK clocking functionality.
	- Triggers for sending new data 
	- Fast recovery
		- Timer instead of DUPAcks
	- 