---
title: 'A Protocol for Interledger Payments'
date: 2019-02-23
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - blockchain
  - distributed ledgers
  - credit network
  - payment channel network
---

# Reading Intention
* Learn how routing is done
* Learn if and how sender/receiver unlinkability is achieved

# Problem
* How can existing (and generally incompatible) distributed ledgers be connected in such a way as to create a currency-independent credit network?

# Assumptions
* Escrow in each ledger is secure:
  * It cannot be fooled into thinking that some condition has been met before releasing funds
  * When the conditions for it to release funds are met, the funds will be released as promised
* The path is chosen by some external means; its calculation is not part of this paper
* Relies on liveness and safety of the underlying BFT algorithm in atomic mode

# Summary
One of the growing problems in the cryptocurrency space is that that are now hundreds of different ledgers and coins in use. While this is nice in the free-market sense that people have a lot of choice, is also has hidden costs harkening back to the earliest human days of disparate currencies. The problem is: how can monetary transfers happen between two parties that may not have a shared currency. Well for a long time, the answer was to go through trusted intermediaries like banks. For fiat currencies, this still makes sense. One of the major selling points of cryptocurrencies however, is that there doesn't need to be any kind of centralized trust. So relying on a trusted single party for transfers between currencies is effectively as bad as trusting a single party for maintaining the value of those currencies.

Interledger offers a solution to this problem. It proposes a protocol wherein payments can happen between parties that do not share a common currency. At a high level, it does this by generating a series of payments between __connectors__ where each connector converts from one currency to another. The first requirement of the connectors is that the first connector can accept the type of currency the the initial payer wants to pay in, the last connector can send the type of currency that the payee wants to receive, and all intermediate connectors form an unbroken chain of valid currency conversions between those two. The only requirement for ledgers (or non-cryptocurrency payment systems) to be plugged into the Interledger protocol is that they support escrowing of funds.

There are two operating modes for Interledger, depending on assumptions that may or may not be made. The operating modes are called **Atomic** and **Universal**. The details of each will be discussed below, but the important thing to know before diving in, is that Atomic mode guarantees __atomicity__ (either all transfers happen, or none of them happen), but in exchange it requires a set a trusted __Notaries__. Universal mode can be used if a set of mutually trusted Notaries cannot be established. The trade off is that some safety guarantees go out the window in Universal mode, although the authors do offer some bounds and economic workarounds.

## Atomic Mode
The first step in understanding Atomic mode is to understand its motivation. When making a series of connected payments, each user along the chain wants as much guarantee as possible that if they send funds somewhere, they will receive at least as much from somewhere else. To ensure this, for a given transfer from A to B with n intermediate connectors, either all payments between connectors must execute, or none of them execute. Currency will lost by at least one participant if only a subset of the transactions complete. In a general Distributed transaction, a great way to enforce this is through the Two-Phase Commit (2PC). The 2PC, however, requires a leader. This is effectively the role that the Notaries serve--they act as a replicated leader that needs to come to its own Byzantine fault tolerant consensus about whether all or none of the transactions should execute. Discovering a mutually trusted set of Notaries is the first step in the Atomic mode protocol. Each participant along the path of the transfer makes public its list of trusted Notaries. A subset of Notaries common between all participants will be selected, if possible. If no such subset exists, or is less than size 4 (minimum size for Byzantine fault tolerance), then Atomic mode is not possible, and the protocol will fall back to Universal mode.

Assuming that selecting a trusted set of Notaries was possible, the transaction begins with a proposal. The sender proposes how much each transfer should be, and each connector verifies that the transfer is possible, the exchange rate is correct, and charges any associated fees. The sender collects these responses, and when all have been received, it begins the prepare phase. To begin the prepare phase, the sender puts the payment it owes the first connector into escrow (with a refund to the payee to occur automatically after a timeout __t__, if a certain condition is not met), and offers some proof of that to the connector. Once the connector has verified that the funds are in escrow, it does the same thing for the next connector viz. it puts the funds __it__ owes into escrow. This continues down the path of connectors. Eventually, the end recipient of the payment will see that the funds it is owed are in escrow.

This is where the Notaries come in. Upon confirming that its funds are escrowed, the end recipient sends a signed confirmation to the Notaries. The Notaries then come to a consensus about if the payment should execute or abort. Usually, the only cause for aborting would be that that timeout __t__ has expired. Whatever the Notaries decide, they send that decision to all participants, and the participants either complete the transaction or abort according to the decision.

All of this talk about trusted Notaries seems to contradict one of the the key claims of Interledger, namely that trusted intermediaries are not needed. The truth is, sometimes trust between parties may exist, and if that's the case, using it can provide optimizations and a safety guarantee. It is however, not strictly required. This brings us to Universal mode, which can be used if there is no such trusted set of Notaries.

## Universal Mode
Universal mode is very similar to Atomic mode. In fact, its entire existence is relevant only to handle one part of the protocol. In Atomic mode, the end recipient sends a signed message to the Notaries that the funds it is owed are in escrow--since there are no Notaries in Universal mode, it can simply execute the transaction, transferring the funds from escrow to itself. The previous connector inline, seeing that funds have been removed from escrow, can now do the same thing. This happens one connector at a time until all payments have been removed from escrow. In this case, all funds must be escrowed on the condition that a receipt signature is provided. This means that a participant can only release the funds it is owed from escrow once it has received confirmation that it's own payment has been released from escrow. There is one complication however. Each escrow transaction has a timeout, and since there is no global guarantee that all transactions will happen, it is possible that a receipt signature could be sent right before the timeout, in which case, the recipient will receive their owed funds, but one of the connectors will pay but not receive anything in return. The paper proposes a couple ways to mitigate the damage and reduce its likelihood. One way to make this situation less likely to occur is to make each of the timeouts different, that is the last connectors to execute should have the longest timeout. The increase of the timeout should include the expected round trip message time and a factor for clock skew. In an asynchronous network like the internet, this does not offer a guarantee per se, but does make partial payments less likely. From an economic perspective, one mitigation could be to increase transfer fees to include the possibility of failure.

# Citation
Thomas, Stefan, and Evan Schwartz. "A protocol for interledger payments." URL https://interledger.org/interledger.pdf (2015).

# Source
[Link to paper](http://blockchainlab.com/pdf/interledger.pdf)
