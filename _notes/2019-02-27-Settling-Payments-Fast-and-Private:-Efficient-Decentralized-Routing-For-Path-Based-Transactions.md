---
title: 'Settling Payments Fast and Private: Efficient Decentralized Routing for Path Based Transactions'
date: 2019-02-27
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
  - privacy
  - payment channel networks
---

# Reading Intention
* Understand type of routing used
* Understand what degree of unlinkability is provided and how

# Problem
* How can payments be routed efficiently in a credit network without centralization and while maintaining value privacy and sender/receiver unlinkability?

# Assumptions
* No public ledger available
* Low topology churn rate
* Adversary controls only a small subset of nodes--specifically, they cannot control all the nodes on a path
* Out-of-band communication is possible between nodes sharing links and between source-destination pairs
* The network does not allow zero-weight bidirectional links to form
* Communication is synchronous: if timeout is too short, some nodes may be prematurely connected unidirectionally--too long and it will be slow to find new routes

# Summary
_SpeedyMurmurs_ is the name of the credit network implementation provided in the paper. This is an extension of the routing algorithm from VOUTE, and the anonymity piece of the _SilentWhispers_ credit network. The main contribution in this paper is their efficient routing algorithm.

## Routing in SpeedyMurmurs
The routing algorithm is broken up into three operations called (in the paper): setRoutes, setCred, and routePay.

### setRoutes Algorithm
Routing in _SpeedyMurmurs_ is a prefix-embedding landmark routing scheme. At a high level, it works by creating a spanning tree connecting all the nodes and assigning an embedding (i.e. address) to each of them. The addresses themselves provide some routing information and will be used in a later phase. In practice, it creates several spanning trees: one for each _landmark_. A _landmark_ is a well known node in the network that is used to find an efficient path. It is similar to a _rendezvous point_ in Tor--it is a location that all nodes how to route to.

The algorithm starts by generating a spanning tree for each landmark. Starting with a particular landmark, it will do a BFS of the tree (at first, only taking bidirectional links into account) assigning embeddings as it goes. The embeddings are more-or-less a list of values where a given node's embedding is the same as its parent, but with a random _b_-bit value appended to the end. The landmark has an empty embedding: (). If after processing all of the bidirectional links, there are some unidirectional links that are not included in the spanning tree, they will be added.

### setCred Algorithm
Sometimes it is necessary to change the weight of a link. Link weight in this context represents how much credit is allotted in one direction. For most link weight changes, it is simply enough to inform both endpoints of the link, however, there are certain situations where greater changes are necessary. If a link has its weight set to zero, meaning that it no longer can be a part of any routes, it will be removed from the spanning tree. Any descendants that it has will need to be re-added to the tree as well. The other situation to consider is if a link is added or in other words: its link weight is increased from zero. In this case, the link will simply need to be added to the spanning trees of each landmark.

### routePay Algorithm
_routePay_ is where the payment logic and actual routing of payments occurs. This operation includes three phases: generation of anonymous return addresses, transaction splitting, and payment routing.

#### Anonymous Return Addresses
The destination sends a tuple that is a keyed hash of the embeddings padded until a fixed length.

#### Transaction Splitting
The total payment is split into c<sub>i</sub> randomly sized pieces which are each sent along a different landmark route.

#### Payment Routing
The routing is done greedily, on-demand. Each node decides which node is the next closest node to the destination.

# Citation
Roos, Stefanie, et al. "Settling Payments Fast and Private: Efficient Decentralized Routing for Path-Based Transactions." arXiv preprint arXiv:1709.05748 (2017).

# Source
[Link to paper](https://arxiv.org/pdf/1709.05748)
