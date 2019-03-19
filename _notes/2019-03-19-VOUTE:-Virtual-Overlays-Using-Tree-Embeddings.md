---
title: 'Voute-Virtual Overlays Using Tree Embeddings'
date: 2019-03-19
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - anonymity
  - p2p
---

# Reading Intention
Understand how they do anonymous addressing and routing.

# Problem
How to route messages with _receiver anonymity_ in an F2F overlay

# Assumptions
* Sender anonymity is not considered--they say it is already a solved problem
* Attacks on availability are not considered--e.g. DoS
* Attackers can not see the full topology, specifically, they cannot be certain that they know all of a nodes connections
* Acceptance of parent invitations in tree-building occurs in rounds, so it is a synchronous process
* Nodes will be honest and forward transactions

# Summary
VOUTE tackles the problem of routing in a Friend-to-Friend (F2F) overlay. It's main contributions are its preservation of receiver anonymity and its efficiency. The efficiency is extracted through the use of _tree embeddings_, and the anonymity is achieved through the strategic use of random numbers and hashing. Both will be discussed separately.

## Tree Embedding
### Forming A Spanning Tree
Routing using tree embeddings is a multistep process. At a high level, it involves creating a tree, assigning "addresses" to each node, and then finding routes based on the addresses. Choosing the root node is deferred to previous work, but in general, relies on some sort of consensus process. Once the root node is selected, a spanning tree is formed. This is done recursively. The root node lets all its neighbors know that it is in the tree now and is available to be a parent--these messages are called _invitations_. Any of these neighboring nodes who do not have this root as a parent in any spanning tree will immediately accept this invitation. If they do already have this node as a parent, then they will hold onto the invitation and at the end of the round, they will decide on a parent from all of the invitations they have received. This end-of-round decision is based on two factors. If one potential parent is this node's parent in less spanning trees than the others, then it will be selected--if this is not the case, then it will be chosen probabilistically. This probabilistic decision-making adds an element of unpredictability, while choosing a parent based on how many times it is already a parent of that node distributes power, so no node becomes overly influential.

### Assigning Embeddings
Once the spanning tree is formed, embedding need to be assigned. These embeddings are the coordinates that will be used for making routing decisions. A node's embedding is just a in-order list of its parent's IDs. The root node has the ID `()`, it's children will have IDs `(C1)`, their children `(C1 C2)` and so on. A node is in charge of assigning it's own ID. It should choose a random number, so that in knowing a particular node's embedding, an adversary will not be able to guess the embeddings of it's siblings.

### Routing
Routes are determined by common prefix length of the embedding. If a node knows that it needs to route to a particular address, then it simply needs to route the message back up the tree until that node's embedding matches the start of the destination address. Then the node can forward the message to the appropriate child--this continues until the message reaches its final destination.

## Anonymous Return Addressing
Return addresses are anonymous to the both sender and all the intermediate nodes along the path. This is done by generating the return addresses in a three step process:
1. Padding the coordinate with random data. This ensures that no node will know how much farther it needs to route a message, with the exception of the destination node, of course.
2. Applying a hash cascade to the embedding. For the first hash in the cascade, a random number is generated and then hashed with the first coordinate in the embedding. Subsequent hashes in the cascade are created by taking the previous hash's output, and hashing it _again_ with the current coordinate in the embedding.
3. A MAC is appended to the end.

This pair, the hash cascade and the random secret key that was generated in step 2, consist the anonymous return address. Every node that receives it will attempt to decode it by applying the same procedure to their embeddings using the same secret key. A node will then be able to determine if a) it is the destination, or b) if it should should forward the request on on to one of it's children.

# Citation
Roos, Stefanie, Martin Beck, and Thorsten Strufe. "Voute-virtual overlays using tree embeddings." arXiv preprint arXiv:1601.06119 (2016).

# Source
[Link to paper](https://arxiv.org/pdf/1601.06119.pdf)
