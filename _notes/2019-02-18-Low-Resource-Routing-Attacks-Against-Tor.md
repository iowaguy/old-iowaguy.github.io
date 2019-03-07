---
title: 'Low Resource Routing Attacks Against Tor'
date: 2019-02-18
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
  - attack
---
[Link to paper](https://www.researchgate.net/profile/Dirk_Grunwald/publication/221342248_Low-resource_routing_attacks_against_TOR/links/00b495224c7e66c6c8000000.pdf)

# Problem
* How can a source and destination be linked in a Tor circuit with minimal bandwidth?

# Reading Intention
* What kinds of routing attacks are out there, and how to think about routing attacks?

# Assumptions
* A circuit can be de-anonymized if an attacker controls both the entry and exit relays
* Circuit creation contains a distinctive pattern of packets that can be identified

# Summary
It is almost disappointing how easy it is to break Tor's anonymity with this attack. The original Tor paper made certain assumptions about how anonymity would be protected based on how the Onion Routers (ORs) were chosen, but when the ORs were chosen in a new way as part of an effor to increase performance, the assumptions--and consequently, anonymity--were broken. This summary describes the state of Tor routing as of the writing of this paper (circa 2007), certain aspects may have changed in the meantime.

## Tor Routing
To understand why Tor is vulnerable, it is important to first understand how its routing works. In the original paper, it was stated that ORs would be selected at random, with only loose restraints like: no OR can be in circuit twice, and the routes must adhere to relays' exit policies. However, in an effort to increase performance, and new routing algorithm was chosen.

This routing algorithm prioritizes stable, high-bandwidth nodes and operates in two stages. A _stable_ node is one that has an uptime greater than the median network uptime. The two stages are: choosing the entry node, and choosing the subsequent nodes.

### Choosing The Entry Node
The reason entry nodes (AKA _guard nodes_) are thought of as different than other nodes is because if entry nodes are rotated as often as other nodes, then an attacker has a greater chance of getting their node selected as the entry node. The entry node learns who is making the initial connection, so increasing the chance that it is an honest node has great benefit. As such, the directory servers have a list of preferred entry nodes. These are a subset of the stable, fast nodes (_fast_ in the sense that they are above the median in bandwidth). After the entry node is chosen from this subset, the remaining relay nodes must be chosen.

### Choosing Non-entry Nodes
Again at this stage, nodes with more bandwidth are more likely to be chosen. This is done probabilistically. The probability that any individual node will be chosen is equal to the percentage that that node's bandwidth makes up of the total network bandwidth. With this heuristic, all nodes will be chosen sometimes, but higher bandwidth nodes will be chosen more often.

## The Attack
The way that the directory servers get bandwidth and uptime information is by advertisements from the nodes themselves. A dishonest node can easily lie about these numbers and thereby change the likelihood that it will be selected. The other option is that an attacker could try to enlist actual stable, high bandwidth nodes to its attack. This second approach will be more difficult, but whichever approach is chosen, the following steps are the same.

When an Onion Proxy (OP) tries to create a circuit, it will determine which nodes to connect to. If the path includes attacker nodes as both the entry and exit nodes, then it's a simple matter of using a timing attack to correlate traffic. If the path does not include attackers at the entry and exit nodes, but instead, only contains one attacker, then that attacker will cause the circuit to break, prompting the OP to find a new circuit. After doing this enough times, eventually the circuit will contain the requisite attackers, and will be vulnerable.

As described, this attack will only work with OPs that have started _since_ the attack began. This is because it is only at startup time that an OP requests a list of entry nodes from the directory server. However, even this can be circumvented. There is one circumstance in which new entry nodes can be requested. That is if the other entry nodes have been tried, and an attack free path cannot be found. The entry nodes are publicly known, so they can be taken down by various DoS attacks. Alternatively, if the circuits are broken enough times, eventually the OP will also request new entry nodes.

# Citation
Bauer, Kevin, et al. "Low-resource routing attacks against tor." Proceedings of the 2007 ACM workshop on Privacy in electronic society. ACM, 2007.
