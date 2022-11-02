For simulating network conditions:
```
CONFIG_NET_SCHED=y
CONFIG_NET_SCH_NETEM=y
```

For TBTCP:
```
CONFIG_TCP_CONG_TB=y
CONFIG_DEFAULT_TB=y
CONFIG_DEFAULT_TCP_CONG="timer_based"
```

