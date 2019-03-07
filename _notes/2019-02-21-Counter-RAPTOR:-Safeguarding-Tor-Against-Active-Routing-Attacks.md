---
title: 'Counter-RAPTOR: Safeguarding Tor Against Active Routing Attacks'
date: 2019-02-21
collection: notes
permalink: /:collection/:year/:month/:day/:title.html
tags:
  - routing
  - security
  - defenses
---
[Link to paper](https://arxiv.org/pdf/1704.00843.pdf)

# Problem
* How can Tor protect user anonymity in the presence of BGP hijacking attacks?

# Assumptions
* Metrics reported are accurate and up-to-date
* The increased risk of fingerprinting is worth the decreased risk of BGP hijacking
* Only guard relays are worth protecting from BGP hijacking

# Opinion
I don't like that they've used the term "resilience." It sounds more like a robustness property. Resilience makes me think of how well something recovers after a failures. Paper did not include a threat model or system model.

# Summary
Tor, as a service that accessible over the internet has proven to be prone to BGP hijacking attacks. This paper presents two methods for mitigating the effects of BGP hijackings. All of this paper's solutions rely on a concept they call _resilience_.

## The Attack
BGP hijacking can be used to direct traffic to an Onion Router (OR) different than that which it was intended. This is in the context of _equally specific_ prefix attacks, which is where the BGP prefix of the true origin and the false origin are the same length. Equally specific attacks are more interesting here, because they are harder to detect. If an attacker uses a shortest path attack, then it will be obvious, because all traffic will be directed to the attacker's more specific BGP prefix. There is also another possibility for an active attacker using an equally specific attack. They can perform an interception attack, which allows them to direct traffic towards their own OR and then forward it on to its intended destination. This can be valuable for traffic analysis.

## Resilience
In this paper's parlance, resilience is used to represent how robust a relay is to being BGP hijacked. It is a number, from 0 to 1, that represents the ratio of traffic routed to the correct OR to traffic routed to the false OR.

## Proactive Defense: Tor Guard Relay Selection Algorithm
One defense is to prevent these types of vulnerable relays from becoming a part of circuits in the first place. Since this is more a matter of ratios rather than a strict classification, the algorithm proposed uses resiliency probabilistically--in a way similar to how bandwidth is used. A node with higher resiliency is more likely to be chosen.

## Reactive Defense: Detection
Routing around vulnerable nodes may reduce the odds that an attack will happen, but it doesn't eliminate them completely. This creates a need for a second line of defense. By detecting attacks in progress, their damage can be mitigated. This detection process has three steps: collecting metrics, detecting anomalies, and mitigating attacks.

### Metrics
There are several public sites where general internet and Tor-specific data can be gathered from. This data includes the IPs of the Tor ORs and BGP announcements.

### Anomaly Detection
1. The BGP data for the Tor relays is compared to known values in a long-running process
2. The frequency of prefixes being proposed is tracked. With this method, inconsistent proposals can be detected
2. If a prefix is announced for only a brief period of time, it may be an attack

### Mitigation
Relays that are in a hijacked domain are blacklisted. Relays that are blacklisted will not be used as guards for some period of time. When the prefix has returned to normal, the prefix will be removed from the blacklist.

# Questions
* section III D, Analyzing low resilience values of ASes, why does the stub AS remain unaffected?

# Citation
Sun, Yixin, et al. "Counter-RAPTOR: Safeguarding Tor against active routing attacks." Security and Privacy (SP), 2017 IEEE Symposium on. IEEE, 2017.
