---
title: 'STREAM: A Multiplexed Money and Data Transport For ILP'
date: 2019-02-25
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - blockchains
  - credit networks
  - payment channel networks
---

# Reading Intention
* Understand how routes between sender and recipient are established
* Learn what types of privacy or anonymity properties (especially sender-receiver unlinkability) STREAM has, if any

# Problem
How to send payments or data through a blockchain _liquidity network_ on an ongoing basis.

# Assumptions
* Links are bi-directional
* Shared secrets are communicated out of band (not through the STREAM network)

# Questions
* What purpose does the condition serve? Does it only designate whether or not a payment is fillable?
* For what purpose would this be used to send data? It borrows extensively from QUIC, but what does it add aside from payments between ledgers?

# Summary
STREAM is a protocol that can be used to send money or data. It is built on the Interledger protocol, and so it routes its payments and messages through existing blockchain payments. The integrity of payments is handled by the Interledger protocol through its use of escrow and _atomic commits_. STREAM is a connection oriented protocol. It consists of two high-level concepts: _connections_ and _streams_.

## Connections
These represent a connection between two specific endpoints (senders and receivers) i.e. there can only ever be at most one connection between any unique pair of endpoints. This is useful because it allows all of the key management work to be done only once for each pair, even if the connection contains multiple streams.

### Connection Migration
Either endpoint of a connection has the option to migrate itself to a different address. It can do this by sending a `ConnectionNewAddress` message. The other endpoint will only heed the change if it receives a valid response from the new address (encrypted with the connection's shared secret).

## Streams
Streams (note: different than STREAM) are instances of ongoing payments or data transfers. A connection may have many _multiplexed_ streams along the same connection. They will all use the same cryptographic material, which reduces storage requirements.

### Exchange Rates
Interledger payments may be subject to exchange rates for going through connections between blockchains. These are unavoidable because they represent the economic incentive for connectors to participate in the Interledger protocol. As such, they need to be calculated as a part of STREAM. This is done at stream creation time. When a receiver requests a payment, they can specify a minimum payment amount. As the message is propagated through the network towards the sender, it collects the exchange rate information from the connectors. The actual amount that the sender ends up sending must be greater than the sum of the minimum payment amount and the exchange rate fees.

## Flow Control
### Stream Level Flow Control
An endpoint of a stream can advertise a maximum amount of data or money it can receive. If this limit is surpassed, a `FlowControllError` will result. If a limit is reached, the sender can send a `StreamMoneyBlocked` or `StreamDataBlocked` message to indicate that it would violate the limit if any more is sent. The receiver can, at this point, increase their advertised limit by advertising a new limit. However, they should be cautious in doing this because they can not later reduce the limit once it has been advertised, except by closing the stream.

### Connection Level Flow Control
This works in the same way as stream level flow control except that it only cares about the aggregate amounts across all streams.

# Source
[Link to protocol](https://interledger.org/rfcs/0029-stream/)
