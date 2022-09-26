# Implementing a congestion control module
This document will detail the implementation process along with general notes on how to implement a pluggable congestion control module in the Linux kernel. 

## TCP in the kernel
TCP is organized in the following files in `net/ipv4`:
	- `tcp.c/h` 

## Congestion control modules
The kernel source is shipped with several congestion control modules. These reside in `net/ipv4` with the naming convention `tcp_<NAME>.c`.  