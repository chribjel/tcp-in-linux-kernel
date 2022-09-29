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

The `.protocol` field references the protocol handler for TCP:

![[Pasted image 20220926151842.png]]

The handler `tcp_v4_rcv`  in `tcp_ipv4.c` is the main driver for TCP as it is called to handle incoming data that is passed from the network layer. This entails both incoming application data, ACKs, and connection requests (SYN). 

### tcp_v4_rcv
This function accepts a socket buffer (`sk_buff` ) that contains the incoming packet. It runs the packet through various tests: it validates the packet type, header sizes, checksum, etc. If any of these fail, the packet will be discarded. 

If the packet passes initial checks, the function will try to fetch a socket to associate the packet to. It does this by accessing hash tables based on the packet header information.

![[Pasted image 20220928121555.png]]

When a socket is found, the packet will be processed based on the socket state (RFC 9293):

- `TIME_WAIT` : means that the host has received an ACK for a  `FIN`  packet. It now waits for the peer to send `FIN`. Its purpose is to close connections gracefully.
- `TCP_NEW_SYN_RECV`: means that a `SYN` was previously recieved and that the socket is now waiting for an ACK for its own `SYN` request. After the packet has gone through some checks, `tcp_check_req` is called, which handles the re-transmission of the `SYN/ACK` or advances the connection state to `ESTABLISHED`.
- `TCP_LISTEN` : means that the socket is waiting for a connection. `tcp_v4_do_recv` is called to handle this.
- `TCP_ESTABLISHED`: represents an open connection, "business as usual" state. `tcp_v4_do_recv` also handles this case.

### tcp_v4_do_recv
This function accepts a socket and a socket buffer to handle either the `TCP_ESTABLISHED` or `TCP_LISTEN` states. 

For `TCP_ESTABLISHED`, something is done to the `dst` demux field, unsure what. After that `tcp_rcv_established` is called to handle the packet.

For `TCP_LISTEN` is different. The packet is run through `cookie_v4_check` to determine if it carries a valid SYN-cookie. If it is, a socket with cookie options is returned (three way handshake should be completed?) , else the socket will remain unchanged. If the socket is unchanged does not have valid cookie options, it is given to `tcp_rcv_state_process` for further handling.

### tcp_recv_established
TCP receive function for the `ESTABLISHED` state. This function will first check if header prediction should be done; check if the packet is 'next in sequence' rather than 'in the window', because the former should be faster. 

Header prediction is done by checking if the header contains HP bits and if sequence number is the next to be received (+ some other things). 

- If the segment length equals `tcp_header_len`, we know that it is an ACK, so we treat it as one, mening `ts_recent` timestamps are stored, the ACK is handled with `tcp_ack` and whether or not to send data is checked with `tcp_data_snd_check`.