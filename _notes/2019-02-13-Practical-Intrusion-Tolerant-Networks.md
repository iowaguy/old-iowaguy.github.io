---
title: 'Practical Intrusion Tolerant Networks'
date: 2019-02-13
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
  - networks
---

[Link to paper](http://www.dsn.jhu.edu/papers/icdcs2016_PITN.pdf)

# Problem
* How to offer message reliability or prioritization guarantees for routing packets in a closed overlay network when the overlay nodes and/or the underlying physical nodes may contain active byzantine faults.

# Assumptions
* Bi-directional links
* assumes that ISP data centers are connected strategically such that each one is part of a different disjoint route
* Overlay topology is stable, does not change frequently
* Some IP networks can fail (but not too many)
* Overlay nodes can become byzantine
* Any node can inject messages into the network
  * can lose, duplicate, misorder, or corrupt messages
* PKI is available and trusted

# Opinion
While a network like this certainly has some use cases, it will probably have too much overhead to be used in most applications. The large space requirements for the MTMW make it unuseable for internet scale applications.

# Summary
The value in Practical Intrusion-Tolerant Networks is in its robustness at two layers: network and overlay. This separates it from Perlman’s Byzantine robust routing, which is at the mercy of the underlying network.

## Network level robustness
This is achieved by making intelligent selections of where overlay nodes should be located. This selection is based on two criteria. The first is that, in the quest to establish disjoint paths in the overlay, the overlay nodes themselves must be connected in disjoint paths with regards to the underlying network. The second relies on using multihoming with multiple ISP networks. It is assumed in the paper that selecting nodes in different ISP data centers will achieve disjointness, because the ISPs themselves won’t be strictly relying on other ISPs for establishing paths. My problem with this assumption, is that, while probably sufficient in the sense of ISPs, it may not be applicable in other contexts that may also wish to establish network level disjointedness.

## Overlay level robustness
Routing in the overlay is achieved using three tools. Maximal Topology with Minimal Weights (MTMW), K node-disjoint paths, and constrained flooding. MTMW is a specific collection of information that is distributed by a system admin. It is provided to each node, and specifies the network topology—who’s connected to who— and what the minimum weight is for each of the links. The link weights can be adjusted based on metrics (e.g. latency or loss rate), but cannot be lowered below the minimum in the MTMW. This approach has the weakness of relying on a centralized disseminator of information (the sys admin). The MTMW qualifies the routing as a link-state algorithm. However, the routing is also source-based, so that an honest sender doesn’t need to rely on intermediate nodes for path decisions.

There are two ways in which a path through the network can be determined. The first is by using a K node-disjoint paths algorithm a la Perlman’s thesis. This guarantees a path will be found as long as there are K-1 or fewer byzantine nodes, given that are K node-disjoint paths from source to destination. Determining disjoint paths can be done with one of several solutions to the max flow problem, such as Ford-Fulkerson. If this method is used, a message is sent along all K paths at the same time, and if the assumption of K-1 or fewer attackers holds, message delivery is guaranteed.

The problem with K node-disjoint paths however, is that if there are K or more attackers, it no longer guarantees that a path will be found. To get around this, constrained flooding is used. As long as a non-faulty path exists between source and destination, it will be found with constrained flooding. Constrained flooding is effectively Perlman’s robust flooding, but it has been slightly optimized to reduce the number of messages sent. The choice of dissemination method can be chosen on a per-message basis, depending on which tradeoffs are most important.

### Messaging protocols
PITN also defines two types of messaging semantics which—backed by different protocols—can offer different guarantees that can be useful in certain situations.

#### Priority Messaging with Source Fairness
Priority Messaging with Source Fairness allows a priority field to be included in the message. If a situation arises where some messages need to be dropped, lower priority messages will be dropped first. This works under the covers by allocating a buffer at each node for each sending source. As messages come in from a certain source, they are buffered in the order that they arrived—with the lowest priority messages being dropped if the buffer is full when new messages arrive. Messages are sent in round robin fashion among all the sources. For Priority Messaging, ordering is not guaranteed. At a high level, Priority Messaging is analogous to UDP, whereas Reliable Messaging is analogous to TCP.

#### Reliable Messaging with Source-Destination Fairness
Reliable Messaging with Source-Destination Fairness has better reliability guarantees for all messages, but can be slower, and therefore may not be sufficient for certain time-sensitive applications like monitoring. For Reliable Messaging, a buffer is allocated on each node for the source-destination pair. As messages arrive, they fill up the buffer (which is concurrently being emptied in round robin fashion for fairness). A message is kept in a buffer until it receives an ack from the final destination.

# Citation
Obenshain, Daniel, et al. "Practical intrusion-tolerant networks." Distributed Computing Systems (ICDCS), 2016 IEEE 36th International Conference on. IEEE, 2016.
