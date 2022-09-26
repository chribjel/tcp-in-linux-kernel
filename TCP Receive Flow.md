# TCP Receive Flow
This document will detail an overview of the TCP receive flow in Linux.

## Sockets
Sockets in Linux are implemented as BSD sockets (Berkeley Sockets) to be used in applications to be an endpoint for communication:

```
int socket(int domain, int type, int protocol);
```

Where `domain` specifies which protocol to be used for communication across hosts. Ex. `AF_LOCAL` (`AF_UNIX`) is used for communication on the same host, while `AF_INET` is defines the IPv4 protocol for communication between different hosts.

The `type` specifies the needed traits of the socket. Ex. `SOCK_DGRAM` for UDP and `SOCK_STREAM` for TCP in Linux.

The `protocol` field specifies the protocol to use to support the `domain`. Usually, only a single protocol is supported per family. Ex. `protocol` 0 is used for `AF_INET` as this signifies IP.

## TCP/IP
When creating an `AF_INET` socket with a `SOCK_STREAM`  type, the socket will be using TCP/IP as its protocol. This is defined in the `inet_protosw` array in `af_inet.c` which is filled on startup. 

![[Pasted image 20220926150756.png]]

The kernel TCP components used in kernel space are defined by the `.protocol` field. Which references the following: 

![[Pasted image 20220926151842.png]]

The key driver function here is the `.handler` which 