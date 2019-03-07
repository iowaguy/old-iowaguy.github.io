---
title: 'Routing Cryptocurrency with the Spider Network'
date: 2019-03-01
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - congestion control
  - payment channel networks
---

# Reading Intention
Look for security issues or unrealistic honesty assumptions.

# Problem
What are some optimal properties that a routing algorithm in a _packet-switched payment channel network_ should have?

# Assumptions
* All nodes are trusted
  * the receiver is trusted because for non-atomic payments, it must report back how much of a payment it has received
  * the intermediate nodes along the routing path are trusted because they must forward payments to the correct next hop

# Summary
This paper has two key contributions. They introduce the idea of _packet-switched routing_ in the context of _payment channel networks_ (PCN), and they provide mathematical proof of some properties that a routing algorithm in such a network should have.

## Packet Switching
The concept of packet-switching in a communication network is not a new idea. It has been a standard procedure for internet routing for some time now. The original idea is that a message can be split up into little pieces which it may be easier to route because of their smaller size. In the context of a payment, this means splitting the payment up into equal-sized _transaction units_. These units can be routed independently, which means that the link weight requirements (i.e. having enough funds to forward a payment) are less.

### Non-atomic Payments
When payments are split into pieces, it is possible to lose the guarantees on payment _atomicity_. The Spider Network offers a solution to this, which will be discussed in the next section. However, there may still be some value in non-atomic payments. A non-atomic payment works by routing the small transaction units from the sender to the destination. After some configurable timeout period, the destination will report back to the sender how much it has received. The remainder of the payment can then be sent in a second payment.

In order for the sender to have more granular control over each transaction unit, it is recommended that they each be encrypted with a different key.

### Atomic Payments
Atomicity can be guaranteed by certain cryptographic means. In particular, a single key can be used to encrypt all of the payments, and then that key can be split up into _n_ segments using a threshold scheme. Only when a destination has received all of the _n_ key segments will they be able to reconstruct the key, and complete the payment.

## Routing
The second contribution of this paper is their mathematical analysis of routing properties. The problem which prompted this work is called _link imbalance_. The idea is that as long a link has available funds in one direction, payments can be sent along it, so if all the funds have been spent in one direction then no other payments can be routed through it, created a more limited topology. This will lead to a network slowdown as payments are forced to be transmitted over fewer channels. A way to counteract this is to make sure that payments are being routed over links at a equal rate in both directions. This will prevent a links funds from becoming depleted in one direction.

There are two routing implementations that were weakly tested. The first, called _waterfilling_ first routes payments along the link with the highest weight, until it becomes tied with the link with the second highest weight. At that point, it splits the payments across both links. Eventually, those two links will become tied with the link with the third highest weight, and then the payments will be split three ways. This process continues until the payment is complete or the timeout happens.

The other implementation, called Spider LP, sets link weights based on an optimization problem (where the total amount of network flow is being optimized).

# Citation
Sivaraman, Vibhaalakshmi, et al. "Routing cryptocurrency with the spider network." arXiv preprint arXiv:1809.05088 (2018).

# Source
[Link to paper](https://arxiv.org/pdf/1809.05088.pdf)
