## Abstract

An implementation and deployment plan for `AltNet` is proposed, a new subsystem aiming towards 
providing a generic framework for integrating alternative transport communication channels inside
Bitcoin Core, hardening the p2p network.

### Design goals

1. Increase network security by increasing link layer/peers diversity
2. Increase transactions anonymity by providing identityless broadcast
3. Increase application security by allowing corrective behavior due to anomalies detection

### Plan

The phase 1 aims to achieve headers-sync between Bitcoin nodes over the Lightning p2p network. Each
Bitcoin node is locally connected to a Lightning node, and the headers should flow over the Noise
communication channel.

There are few building blocks required :
1. a `rust-multiprocess` library to communicate with Bitcoin Core in a multiprocess setup
2. a `Validation` interface to access the validation engine
3. a `altnet-orchester` daemon to serve as a C2C for the alternative transport driver
4. a `altnet-lightning` daemon to connect to a Lightning node
5. few hacks in LDK/LDK-sample

### Ressources

- Github issue: https://github.com/bitcoin/bitcoin/issues/18989
- Poc PR "Lightning sync": https://github.com/bitcoin/bitcoin/pull/18988
- Poc PR "Watchdog": https://github.com/bitcoin/bitcoin/pull/18987

## Basics

### Why alternative communications for the Bitcoin p2p network ?

The current networking approach suffers from a wide range of issues with regards to
[transaction origin inference](https://github.com/bitcoin/bips/blob/master/bip-0156.mediawiki),
counter-measures against [key infrastructure attackers](https://www.comp.nus.edu.sg/~kangms/papers/erebus-attack.pdf),
and [harder security assumptions](https://lists.linuxfoundation.org/pipermail/lightning-dev/2019-December/002369.html)
of higher-level protocols. Being heavily optimized and at same time trying to solve diverse goals
like reasonable network security , tx-relay privacy, hindering block-topology, peer diversity, ...
a functional networking stack is likely unable to address aforementioned issues without [compromising](https://github.com/bitcoin/bitcoin/pull/17332)
its [robustness](https://bitcoin.stackexchange.com/questions/81503/what-is-the-tradeoff-between-privacy-and-implementation-complexity-of-dandelion/81504#81504).

Ideally you want to address scenarios with higher security requirements like an exchange receiving
headers-over-DNS to detect tip pinning. A conscientious user always relying on tx-over-radio for
each of its coins sends. You can also think about a LN hub willingly to use a HTTPS connection to a
block explorer for emergency tx broadcast or an SPV wallet receiving filters-headers-over-obfs4 to
defeat local Internet censorship. Of course, you can still rely on external modules or project, but
better integrating them with Core to ease deployment and combine them for increased benefits.
