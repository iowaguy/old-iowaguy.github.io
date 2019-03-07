---
title: 'Tor: The Second Generation Onion Router'
date: 2019-02-15
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
  - anonymity
---
[Link to paper](https://www.usenix.org/legacy/event/sec04/tech/full_papers/dingledine/dingledine_html/)

# Problem
* How can the unlinkability of TCP connections be protected in a low-latency, onion routing overlay?, and how can it be done in a way that achieves the usability necessary for widespread adoption?

# Reading Intention
* Notice what mechanisms are employed to protect anonymity. What attacks are averted, and how?
* Notice how routing decisions made in Tor.
* What anonymity properties does it claim, and how does it achieve those?
* Differentiate between main mechanisms and performance optimizations

# Assumptions
* An adversary can:
  * See some fraction of the network traffic, but not all of it. Specifically, unlinkability does not hold if the adversary can listen on links to the entry guard and links from the exit nodes within the same circuit
  * Generate, drop, replay, delay, or modify traffic
  * Operate onion routers of their own
  * Compromise some of the network's existing onion routers
  * Use timing to break anonymity if listening at both ends of route, and traffic is distinct enough
  * _Directory servers_ and local _Onion Proxies_ are trusted entities. Directory servers are trusted to provide an accurate reporting of the Onion Routers available
* Prevents _traffic analysis_ attacks, but not _traffic confirmation_ attacks
  * An adversary does not have the ability to view encrypted traffic, and determine where it came from and where it's going to, unless it is listening at both ends of the circuit
* All participants agree on the set of directory servers
* If there is a directory server issue, we can fall back to a trusted human administrator

# Summary
One of the problems in providing a short summary of Tor is that it has many complexities which it uses to protect itself from a slew of attacks. To best capture these complexities, Tor is best explained from the ground up

## Cells
A _cell_ is what Tor calls packets in its overlay. A cell has a specific byte layout, depending on what type of cell it is. There are two types of cells: _control cells_ and _relay cells_. Though all cells have the same size, so that after encryption, they cannot be differentiated by a listening attacker.

### Control Cells
Control cells represent the control plane. When received, they inform the receiving node of the action that the sending node wishes to perform. Control cells have the ability to create new streams, or destroy old streams. A stream is effectively a single TCP connection to some destination, and its flow of data.

### Relay Cells
A relay cell is the type of cell that carries an actual data payload. It also has the ability to extend a circuit by one hop or truncate it.

## Circuits
A circuit is a path through a series of Onion Routers (OR) from the message sender to the message receiver. Since layered encryption is used, an OR only knows the previous and next hop in the route. Therefore, it will only be able to determine that it is the first hop if a request comes from an Onion Proxy (OP), and it will only be able to tell that it is sending to a destination if when it finally needs to make a connection outside the overlay network. As long as there are at least three ORs in the path, no OR will be able to know both the sender and the receiver.

Circuits are built incrementally. As a route to each successive OR is established, a key exchange is performed (more on that below), and when it is complete, another OR can be added to the circuit by the sender's OP, if desired. Circuits are also built preemptively in the background. This provides smoother transitions for streams that were operating on a circuit that needs to be closed down. It takes on the order of tenths of a second to create a new circuit, which is fast if done occasionally, but slow if done frequently. For this reason, new circuits are generated every few minutes.

### Leaky Pipe
There is a concept used to increase anonymity called _leaky pipe_ topology. This simply means that some messages from a circuit can bypass the last OR and connect directly to the service. This makes it harder for a passive observer of an exit node to violate unlinkability.

### Encryption
Layered, or onion encryption, is used for end-to-end encryption. This means that the source will wrap its messages in successive layers of encryption, each of which will be removed by the OR in the circuit that has the corresponding key. This ensures a few things. First, messages must reach the ORs in the prescribed order, or the ORs wouldn't be able to decrypt the message. Second, message never needs to exist as plaintext anywhere other than the source and destination--no one in between will be able to decrypt it unless they have all of the keys.

The keys that are used are symmetric keys, shared only by the source and one OR. Each OR has a different key for each circuit, and no two ORs share the same key. The keys need to maintain perfect forward secrecy, so they are generated at circuit creation time using Diffie-Hellman. It's important to note that when a message leaves the final OR, it may be in plaintext, unless the sender has encrypted it with a key that it shares with the destination. This is application dependent--Tor has no say here. These symmetric keys are called _short term onion keys_. They are only used for a given circuit, and when that circuit is closed, the keys are discarded.

There is also a notion of a _long term identity key_. These are asymmetric public key pairs that are used for signatures and key exchange. These are supposed to be kept long term, but can be changed if required.

## Onion Routers vs. Onion Proxies
Each node that wishes to send messages in the Tor network must have a local Onion Proxy. An OP is a local SOCKS proxy that serves as a single interface to the Tor network. It is in charge of helping the node set up new circuits and get rid of old circuits. It also handles the encryption. This is the only piece of software necessary for a node to send messages in the Tor network.

## Congestion Control
Since this overlay is designed to handle real world routing situations, it must have a way to gauge the current network usage and throttle the link throughput, if necessary. This is done through the tracking of _packaging windows_ and _delivery windows_. This is a fancy way of stating that the amount of bytes in should be roughly equal to the amount of bytes out over a given time frame. In practice, this means that if messages aren't going out of an OR fast enough, then fewer messages will be allowed to enter. At the source of the stream, this may mean that certain senders may have to find another circuit with more available bandwidth.

## Location Hidden Services
It may be desirable for a service to have its location (and therefore, IP address) hidden from public view. Tor allows nodes to connect to a service without knowing its IP. This can be done through the use of _introduction points_ and _rendezvous points_.

### Introduction Points
An introduction point is an OR advertised by a service that be contacted by a user of the service to let the service know that it wishes to connect. The introduction points of a service will be publicly known. When a service creates introduction points, it does so by creating a circuit that leads to that OR, that way the introduction point doesn't know where the service is, and neither does a node connecting to it. The introduction point will not have any actual payload bytes flowing through it.

### Rendezvous Points
Rendezvous points are used by the client connecting to the location hidden service. A client that wishes to connect will create a rendezvous point, by creating a circuit to some OR--any OR but ideally three hops away at least, if the client wishes to protect its anonymity. Then the client connects to one of the service's introduction points, and lets it know that it has created a rendezvous point and provides the location of that OR. At this point, the service has some discretion to decide if it want to allow the client to connect or not. If it doesn't, the request can be dropped with no anonymity lost. If it does want to allow the client to connect, it can make its circuit connection to the rendezvous point. This creates a complete circuit from the client to the location hidden service along which data can be sent. Since intermediaries are used, neither of the end points needs to know exactly where the other endpoint is.

## Exit Policies
One of the concerns of Tor is maintaining an proper economic incentives to promote adoption. It is a reasonable fear of anyone hosting an OR that they could be implicated in a crime or falsely associated with some party because a connection to some service happened through them. This type of connection may discourage some nodes from becoming relays at all. Exit policies allow a node to decide what connections are allowed to made from that OR. Some exit policies stipulate that an OR can only connect to other ORs, this would make it just a mid-level relay. A fully open exit policy would allow that node to connect anywhere, in which case it would largely be used as an exit relay. Being an exit relay is not for the faint of heart. Connections, and possibly plain text messages are sent from their node to any arbitrary website. It is not uncommon for exit relay hosts to be raided by law enforcement. Exit policies can also stipulate exactly which websites it can or cannot connect to. That way an honest, good-Samaritan, OR host can allow anonymity while avoiding the legal complications of connecting to malicious or illegal sites.

## Directory Servers
In order to interact with the network. A OP must know where the OR are. It can learn of these through directory servers. These servers maintain an up to date list of trusted ORs in the network. ORs are required to send periodic keep-alive messages to the directory servers. Directory servers are maintained by trusted administrators and require administrative action to add new ORs. This is to prevent sybils. It is possible that a directory server could be corrupted by a dishonest administrator, so to ensure that only correct ORs are advertised, a threshold of directory servers must approve of any given OR.

# Citations
* [1] Syverson, Paul, R. Dingledine, and N. Mathewson. "Tor: The second-generation onion router." Usenix Security. 2004.
* [2] https://gitweb.torproject.org/torspec.git/tree/path-spec.txt
