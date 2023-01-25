To let the TCP stack handle pacing, the flag `SK_PACING_NEEDED` can be set in the socket (`sk->pacing_status)`. 

## Non-paced Reno

![[Screenshot from 2023-01-24 13-20-07.png]]
The non-paced version is clearly bursty. Not only in slow start but through the whole transfer.

## Paced Reno

![[Screenshot from 2023-01-24 13-20-40.png]]
