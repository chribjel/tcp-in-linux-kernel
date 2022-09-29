Delay and packet loss can be simulated by setting constants in `tc qdisc` for a given interface.

The kernel must be compiled with `NET_SCHED` and `NET_SCH_NETEM`.

Delay can be set like so:
```
tc qdisc add dev eth0 root netem delay 50ms
```

For packet loss:
```
tc qdisc add dev eth0 root netem loss 5%
```

For both:
```
tc qdisc add dev eth0 root netem delay 50ms loss 1%
```

