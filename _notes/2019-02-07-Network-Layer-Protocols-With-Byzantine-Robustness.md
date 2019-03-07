---
title: 'Network Layer Protocols With Byzantine Robustness'
date: 2019-02-07
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
---

[Link to paper](https://dspace.mit.edu/bitstream/handle/1721.1/14403/20150169-MIT.pdf)

# Problem
* How to send a single packet at the Network Layer, using Link State, through a network with an _a priori_ unknown topology when there are active byzantine faults between the source and destination.

# Assumptions
* bi-directional links
* links are not shared
* delivery not guaranteed over a physical link
* must be used on a single physical network
* assumes trusted source and destination
* Can only send one packet at a time
* simple faults can occur where a node stops and then later restarts
* faults can be byzantine
* byzantine nodes can drop, replay, edit, incorrectly forward, or do anything else they want to a packet they receive
* encryption cannot be broken
* faults can occur on the links
  * can lose, duplicate, misorder, or corrupt messages
* a Network Layer is robust to byzantine faults if, with a high probability, it delivers messages between honest sources and destinations given that a non-faulty path exists between them

# Opinion
This is an extremely well written paper. Many details are covered in a clear, logical way. The biggest reason I see for this algorithm's dearth of implementations stems from its requirement to store the entire network topology. That would require too much storage for internet scale routing.

# Summary
## Robust Flooding
Robust Flooding is an algorithm suggested by Radia Perlman in her dissertation Network Layer Protocols With Byzantine robustness. The motivation is that sometimes flooding, though generally inefficient, is the best way to traverse a network, and there may be cases where some of the network nodes are byzantine—that is, they may deviate from the protocol in ways that are (or seem) malicious. This is in addition to the possibility that they may fall victim to simple failures, where the node simply stops responding at all.

At a high level, Robust Flooding works by using public key pairs to ensure fairness and using sequence numbers and redundancy mechanisms to guarantee that unique packets are forwarded throughout the network.

To understand the fairness solution, it is important to first understand the situation in which unfairness can arise. It is assumed that a given router in the network has some amount of buffer space it can use for when the data link is in use, and has to block until it is ready to send another packet. Since a router is a physical device, it cannot have infinite memory to use for buffering. This means that there’s some limit to how much it can buffer. If that limit is reached, then the router will be forced to drop additional incoming packets. If there is a single buffer that is shared between all the possible destinations, then it becomes random chance which messages actually make it on to the buffer before it fills up and has to block. It is possible that by random chance, a single source could have all of its packets dropped, and would, therefore, be unable to communicate. The interesting thing about this pitfall, is that can occur without any Byzantine or even simple failures in the network. The solution to this problem posed by Perlman is to allocate a certain amount of buffer space for each node in the network. With dedicated buffer space allocated, it is guaranteed that at least of some of the packets being sent will make it through the network.

The question now becomes, how can we ensure that we correctly allocate buffer space for all the nodes. This can be done using a set of so-called, “trusted nodes.” The identities of the trusted nodes is known at startup time by all the nodes in the network. When a node starts up, it contacts a trusted node who returns to it a list of identity to public key mappings. For each mapping it receives, it can allocate buffer space. This also means that even if a trusted node behaves maliciously and includes extra identities that are not actually part of the network, the worst thing that will happen is that some extra space will be allocated that won’t be used. If a malicious node leaves out certain identities, this will also not be a problem as long as at least one honest trusted node supplies that identity. The public keys are used to verify the signature on the incoming packets, so that it can verify that a packet from a particular source actually came from that source—this prevents a Byzantine node from using a different node’s allocated buffer.

To ensure fairness among packets that have made it into buffers, a round robin strategy is used to select the next packet to put on the link.

The second important concept is the redundancy mechanism to ensure that a packet packet gets flooded at least once. This where the sequence numbers on each packet come into play. A single source should be using a monotonically increasing sequence number—though there is one situation where that may not be the case, it will be discussed later.

## Robust Link State Routing
At its core, this solution is a link state source routing algorithm. This means that every node needs to know the entire topology of the network. Topology information is spready in the form of Link State Packets (LSPs). Every node in the network will use robust flooding to spread its LSPs. Upon receiving an LSP from every node in the network, the receiving node is able to generate an accurate topology. Each node is resposible for generating its own LSP. Its LSP will contain a list of all the nodes that are directly connected neighbors to the LSP-generating node. The way a node determines who its neighbors are is by each node sending a Neighbor ID message to all its neighbors at startup time. Neighbor ID messages will not be forwarded by an honest node.

This is in the Source Routing class of algorithms so the source is in charge of generating the entire route. It does this by finding a solution to the Max Flow problem. There are many Max Flow algorithms that are acceptable here. The key criterion is that whatever algorithm is used, it must generate _K_ node-disjoint paths. By using _K_ node-disjoint paths, there is a guarantee that a path will be found as long as there are less than K Byzantine nodes.

There are cetain special cases that must be addressed, however. A Byzantine "trusted node" could generate fake identities along with their key pairs, and then generate fake LSPs for all of them. This would make it difficult for any honest node to find a fault-free route through the network. This can be solved, by remembering our assumption that at least one trusted node is correct. In that case, we can find the _K_ node-disjoint paths for the view proposed by each trusted node. As long as one of those trusted nodes is correct, a path without sybils could be found.

If there are _K_ or more Byzantine nodes, a fallback plan is required. Perlman suggests the following three strategies:
1. Assume a strong network model, as long as the model holds, the algorithm will hold.
2. Fallback to robust flooding. This is guaranteed to find a route if one exists, but it is costly.
3. Detect faults and then send LSP updates for the new topology.


# Other links
* [A pretty good summary](https://courses.cs.washington.edu/courses/csep521/07wi/prj/matthew.pdf)

# Citation
Perlman, Radia Joy. Network layer protocols with byzantine robustness. Diss. Massachusetts Institute of Technology, 1988.
