---
title: 'ODSBR: An On Demand Secure Routing Protocol Resilient To Byzantine Failures'
date: 2019-02-10
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
---

[Link to paper](https://dl.acm.org/citation.cfm?id=1341892)

# Problem
How to route packets on-demand when there are active byzantine faults between the source and destination, with the guarantee that a fault-free path will be found if one exists.

# Short Summary
This paper provides a protocol for correctly routing messages in the face of active Byzantine failures. It uses on-demand source routing to achieve this. The protocol consists of three concepts: routing around failures, failure detection, and link weight management.

# Assumptions
* all links are bi-directional
* assumes some degree of reliability at the physical layer
* network is not open
* source and destination are trusted
* does not consider resource consumption attacks by sending a bunch of unauthenticated messages
* strong byzantine adversary
* simple faults can occur where a node stops and then later restarts
* resistant to wormholes, blackholes, and flood rushing
* Can send more than one datagram at a time
* there is a trusted CA
* links can be shared
* encryption cannot be broken
* faults can occur on the links
  * can lose, duplicate, misorder, or corrupt messages
* does not consider sybil attacks

# Summary
## Intro
Routing protocols with Byzantine robustness have been studied extensively since the late eighties, starting with Radia Perlman's thesis [1]. However, with the rise of wireless networking, _ad hoc_ networks became an important area of study. An ad hoc network is one in which the topology changes frequently and two nodes that may wish to communicate may not be directly connected, but may have a path of intermediate nodes which, as a daisy chain, connect the two end nodes. A popular approach to routing in these ad hoc networks is on-demand routing--first proposed for ad hoc networks in DSR [2], but the DSR protocol is not Byzantine fault tolerant. This is where ODSBR (this paper) comes in. In ODSBR [3], a protocol is proposed in which on-demand routing can be made to be robust to Byzantine faults.

## Network Model
It is assumed that a certain number of packets may be lossed due to any type of failure--this is bounded by τ. If the loss rate is greater than τ, a fault is considered to have occured. Note that just because a loss rate is above τ does not mean that the fault was Byzantine--in this sense, all faults will be routed around. It is also assumed that all links in the network are bi-directional. In this model, a Byzantine node can inspect any packet that comes to it, and it can arbitrarily forward or drop any packet it wishes.

## The Protocol
ODSBR consists of three parts: fault avoidant routing, fault detection, and link weight management. Each will be discussed.

### Route Discovery With Fault Avoidance
The robustness of each link is maintained in _weights lists_. Each node tracks the weights of all the links in the network. If it doesn't directly know about a link, it assumes it weight is 1. A weight of 1 means that the link is operating correctly. If a fault is detected, the link weight is increased. The goal of the routing algorithm is to find the least weight, fault-free path between the source and destination. This protocol uses _source routing_--meaning it provides each packet with the entire path it should traverse. If, when sending a packet to a destination, the source does not yet know if a path to that destination, it must discover a new path. It does this by first flooding the network with a REQUEST packet. Upon receiving a REQUEST for a source, _S_, to destination, _D_, delivery, the receiving node sums the link weights it took to get to that point and checks whether or not it has received this S-D pair before. If it has not seen it before, then it will broadcast this new packet to be flooded. Otherwise, it will drop the packet because it has already seen it. When the source sends a request, it will sign the packet, so that it cannot be forged.

Eventually, the destination will receive this request, and upon doing so, it will create a RESPONSE packet which it will send back to the source along the same route it came from--this is where the bi-directionality assumption is important. When a node receives a response, it will check if it has already received the same response before. If it has, it will sum the link weights to that point and if the summed weight is less than the previous summed weight, it will broadcast that new packet to the next link. If it hasn't seen that packet before, it will also sum the link weights and broadcast it to the next link. Otherwise, it will just drop it because it has already seen a better route. Before broadcasting, each node will append its signature to the end of the packet, so the in-order path cannot be forged.

When the source receives the response, it performs the same verifications as the intermediate nodes. It only needs to store the lowest weight path. When that has been discovered, it can forward future packets along that path.

### Fault Detection
It is possible that a link could remain fault-free long enough to return valid responses, so the protocol must also have the ability to detect faults when the occur on a previously trusted path. ODSBR can detect a bad link after log _n_ faults, when _n_ is the number of links on the path. It does this by using _probe nodes_ or just _probes_. A probe is an intermediate node along the path that has been asked to return an ack to the source when it receives a data packet. The list of probes is included in the data packets, so fault detection cannot be separately filtered from the data stream. When the tolerance threshold τ is surpassed, a fault is considered to have occured, and the interval along which the fault occured is split in half with the node in the middle serving as the probe for future packets. As faults continue to occur, the faulty intervals (which are determined by which probe acks are received) continue to be split in half and new probes added. Eventually, using this binary search, a single faulty link can be detected. Note that the fault _node_ cannot necessarily be detected, only one of its links. This is because it will be unknown whether a node refused to forward a packet it received, or refused to send one initially. Probes are slowly retired by use of a counter. When a probe is created, it is initialized with a counter that is proportional to the number of packets dropped and the tolerance threshold τ. Each time it returns an ack, its counter is decremented. The probe is removed when the counter reaches zero.

### Link Weight Management
When a link is deemed faulty, it's weight is doubled. A counter for each link is also tracked just like with the probes. When the counter returns to zero, the weight is halved.


## Conclusion
By using a combination of robust flooding and fault detection, ODSBR is able to build a map of high risk links and route around them. It is able to do the fault detection in log _n_ time, and has a built-in mechanism for healing links who begin to follow the protocol.

# Bibliography
[1] Perlman, Radia Joy. Network layer protocols with byzantine robustness. Diss. Massachusetts Institute of Technology, 1988.

[2] Johnson, David B., and David A. Maltz. "Dynamic source routing in ad hoc wireless networks." Mobile computing. Springer, Boston, MA, 1996. 153-181.

[3] Awerbuch, Baruch, et al. "An on-demand secure routing protocol resilient to byzantine failures." Proceedings of the 1st ACM workshop on Wireless security. ACM, 2002.

[4] Eriksson, Jakob, Michalis Faloutsos, and Srikanth V. Krishnamurthy. "Routing amid colluding attackers." Network Protocols, 2007. ICNP 2007. IEEE International Conference on. IEEE, 2007.
