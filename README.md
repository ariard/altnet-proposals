## Abstract

An implementation and deployment plan for AltNet is proposed, a new subsystem aimed towards providing a generic framework for integrating alternative transport communication channels inside Bitcoin Core.

## Design goals

1. Increase network security by increasing link layer/peers diversity
1. Increase transactions anonymity by providing identityless broadcast
1. Increase application security by allowing corrective behavior based on anomaly detection

## Why alt-communications channels should be supported in Core ?

There are already projects in the Bitcoin ecosystem that benefit from alternative transport protocols. They rely on running a modified Bitcoin Core, which is cumbersome for the alternative transport maintainer and requires an extension of user trust without the Bitcoin Core development and review process guarantees. Integrating alt-communications in Bitcoin Core would 1) ease deployment 2) enable composability 3) enable traffic assignment 4) allow for driver component reusability. For example, instead of creating a new UDP flow or address management from scratch, others can simply reuse a driver library.

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

### Why not integrate alt-coms in net.h ?

Beyond refactoring critical-path code, alt-coms won't fit neatly at the net or processing levels. At the net level, our network stack relies on 1) Berkeley socket model 2) TCP socket 3) peer policy on Internet topology (ASN/IP prefix). At the processing level, the link may be unidirectional or bandwidth-limited and, therefore, won't meet common user expectations.

### What should be the architecture ?  Threads vs. processes ? Multiple daemons ?

For fault-tolerance, a separate process seems like the best option. Any bug in a low-level driver, even something as bad as Remote Code Execution, could be mitigated at the kernel level. A driver-framework would
be flexible by making each driver a daemon in itself using fork() behind the interface. The master process could be multi-threaded to separate data reading/writing from processing.

### What should the interface be ?

A new `interfaces::Altnet` would be introduced, consumed by a new thread `AltnetHandler`. This would take advantage of the new validation queue from https://github.com/bitcoin/bitcoin/pull/18963.

### What are the internal components ?

A high-level RPC user-interface could 1) manage drivers state 2) assign traffic to drivers 3) push txs to a targeted-driver. Fetching logic would connect directly to the drivers-interface and process protocol messages from it. Broadcast logic would propagate transactions, but not relay them.


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

For security reasons, they would be statically linked.

## Review Process

### How to avoid a review iceberg ?

It could take a long time to get something functional merged. Therefore, we could split the  review into temporary non-useable chunks like the `multiprocess` project.

## Drivers-application

### How will drivers be integrated ?

Each driver implementation will bind to an abstract class `CDriver` mapping to generic operations.

## Drivers-interface

## Drivers-libs

### What can be reused across driver implementations ?

Peer management and message compressing could be implemented as libraries.

## Build system

## Testing

## Watchdog

### What is the purpose of watchdog ?

Watchdog reacts to system faults and takes corrective behaviors. It monitors the network, mempool, and block issuance for anomalies and triggers events for internal or external application consumption. i.e., if blocks are not detected for an hour, a LN node may preemptively close its channels.

## Fetching Logic

### What fetching logic is there to implement ?

If this new p2p stack is partially designed to serve in the emergency case, it should be as robust as possible. [Reverse-headers sync](https://en.bitcoin.it/wiki/User:Gmaxwell/Reverse_header-fetching_sync) seems like the right fit.

### What if we receive a longest-valid headers chain from a unidirectional channel ?

We should send `INV(MSG_BLOCK)` to our bidirectional block channels to move the chain tip forward.

## List of Altcoms

- [Meek](https://gitweb.torproject.org/pluggable-transports/meek.git/)
- [Snowflake](https://gitweb.torproject.org/pluggable-transports/snowflake.git/)
- [I2P](https://geti2p.net/en/)
- [Lightning transport protocol](https://github.com/lightningnetwork/lightning-rfc/blob/master/08-transport.md)
- Amateur radio
- Bluetooth
- WiFi
- LoRa
- [Headers-over-DnS](https://github.com/bitcoin/bitcoin/pull/16834)

XXX: sort alt-coms by properties
 
## Resources

* [Erebus paper on the state of key infrastructure attacks](https://www.comp.nus.edu.sg/~kangms/papers/erebus-attack.pdf)
* [On the complexity of integrating tx-relay privacy-preserving enhancement](https://bitcoin.stackexchange.com/questions/81503/what-is-the-tradeoff-between-privacy-and-implementation-complexity-of-dandelion/81504#81504)
* [On improving Bitcoin Network Resilience through Weak-Radio Signals](https://stanford2017.scalingbitcoin.org/files/Day2/Weak-Signal-Radio-Communications-for-Bitcoin-Network-Resilience.pdf)
