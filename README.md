## Abstract

An implementation and deployment plan for `AltNet` is proposed, a new subsystem aiming towards providing a generic framework for integrating alternative transport communication channels inside Bitcoin Core.

## Design goals

1. Increase network security by increasing link layer/peers diversity
1. Increase transactions anonymity by providing identityless broadcast
1. Increase application security by allowing corrective behavior due to anomalies detection

## Why alt-communications channels should be supported in Core ?

Ecosystem-wise, there are already solutions to benefit from alternative transport protocol. They actually rely on running a modified Bitcoin Core which is cumbersome for both alternative transport maintainer and extend user trust to them, without the guarantees of Bitcoin Core development process. Integrating alt-communications in Core would 1) ease deployement 2) enable composability 3) enable traffic assignement 4) help drivers components reusability. Instead of each of them recoding its own udp flow or address management they may reuse a driver lib.

## Topics

- Architecture
- Review Process
- Drivers-application
- Drivers-interface
- Drivers-libs
- Build system
- User interface
- Testing
- Watchdog
- Fetching Logic
- Broadcast Logic
- List of Altcoms
- Resources

## Architecture

### Why not integrating alt-coms in net.h ?

Beyond a cumbersome refactoring on some critical-path, it won't fit both at the net level and processing level. At the net level, our network stack relies on 1) Berkeley socket model 2) TCP socket 3) peer policy on Internet topology (ASN/IP prefix). At the processing level, the link may be unidirectional or bandwidth-limited and therefore may not fulfill common expectations.

### What should be the architecture, thread-vs-process, multiple daemons ?

For fault-tolerance, a separate process seems the best option. That way any bug in low-level driver, even worst-one like Remote Code Execution should be mitigated at the kernel-level. Driver-framework may
be made flexible enough such that each driver is a daemon in itself by fork() behind the interface if necessary. The master process may be multi-thread to seperate data reading/writing from processing.

### What should be interfaces with codebase ?

A new `interfaces::Altnet` may be introduced, consumed by a new thread `AltnetHandler`. We may reuse the new validation queue from https://github.com/bitcoin/bitcoin/pull/18963.

### What should be internal components ?

At the high-level, you may have a high-level RPC user-interface to 1) manage drivers state 2) assign traffic to drivers 3) push tx to a targeted-drivr. Fetching logic would be directly connected to drives-interface and process protocol messages from then. Broadcast logic would do transaction propagation but not relay.


```


          _____________________               _____________
         |                     |             |             |
         |  interface::Altnet  |             |     RPC     |
         |_____________________|             |_____________|
                   |                            |      |
                   |_______________             |      |
                                   |            |      |
                           ________|________    |    __|_______________
                          |                 |   |   |                  |
                          |  FetchingLogic  |   |   |  BroadcastLogic  |
                          |_________________|   |   |__________________|
                                          |     |      |
                                          |     |      |
                                        __|_____|______|_____
                                       |                     |
                                       |  Drivers-interface  |
                                       |_____________________|


```



### Should drivers be statically or dynamically-loaded ?

For security reasons, statically linked.

## Review Process

### How to avoid a review iceberg ?

First chunk before to get _something_ functional may be quite consequent. Therefore review may be splitted in temporary non-used chunks like `multiprocess` project.

## Drivers-application

### How drivers may be integrated ?

Each driver implementation should bind to some abstract class `CDriver` mapping to generic operations.

## Drivers-interface

## Drivers-libs

### What can be reused accross driver implementation ?

Peer management or message compressing may be implemented as libraries.

## Build system

## Testing

## Watchdog

### What the watchdog purpose ?

Monitor network, mempool or block issuance anomalies and trigger events to be consumed internally or by an external application. Thus reacting to system faults to take corrective behaviors. I.e if blocks aree detected for not being received since a hour, a LN node may preemptively close its channels.

## Fetching Logic

### What fetching logic to implement ?

If this new p2p stack is partially designed to serve in emergency case, it should be as robust as we can. Therefore we may implement [reverse-headers sync](https://en.bitcoin.it/wiki/User:Gmaxwell/Reverse_header-fetching_sync).

### What if we receive a longuest-valid headers chain from a unidirectional channel ?

We should send `INV(MSG_BLOCK)` to any bidirectional block channels to move chain tip forward.

## List of Altcoms

- [meek](https://gitweb.torproject.org/pluggable-transports/meek.git/)
- [Snowflake](https://gitweb.torproject.org/pluggable-transports/snowflake.git/)
- [I2P](https://geti2p.net/en/)
- [Lightning transport protocol](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md)
- amateur radio
- Bluetooth
- WiFi
- LoRa
- [headers-over-DnS](https://github.com/bitcoin/bitcoin/pull/16834)

XXX: sort alt-coms by properties
 
## Resources

* [Erebus paper on state of key infrastructure attacks](https://www.comp.nus.edu.sg/~kangms/papers/erebus-attack.pdf)
* [On complexity of integrating tx-relay privacy-preserving enhancement](https://bitcoin.stackexchange.com/questions/81503/what-is-the-tradeoff-between-privacy-and-implementation-complexity-of-dandelion/81504#81504)
* [On improving Bitcoin Network Resilience through Weak-Radio Signals](https://stanford2017.scalingbitcoin.org/files/Day2/Weak-Signal-Radio-Communications-for-Bitcoin-Network-Resilience.pdf)
