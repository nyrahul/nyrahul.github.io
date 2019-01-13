---
layout:     post
title:      Art of WSN Simulation for mesh networks
date:       2017-12-8
summary:    WSN simulation is an important aspect for IoT network protocol or application development. I ll talk about existing simulation techniques, pit-falls, and then introduce Whitefield.
categories: WSN
---

Wireless sensor network simulation plays an important role in development and
deploying wireless networks.
- for systems development, it helps to verify whether the systems assumptions
and protocols will behave in the estimated way.
- for networks deployment, it helps to understand whether the networks will
scale to the estimated range and whether the desired throughput rate and
latencies could be achieved.

WSN based on low-power RFs such as 802.15.4, aka LoWPAN, is considered for this
article. Having said that, most of the article is generic enough and is
applicable to any wireless technology.

For sceptics, I agree that simulation is not the final answer, but it helps
solve most of the issues before the final deployment. And if issues could be
found in the simulation stage, it saves a lot of cost.

Secondly, not all deployments could be tested with hardware from day one. In
one of my works, the requirement was to setup a 1000 node network with 16 hops
and had to deliver performance expectations from this network before the actual
development work could be started. Clearly putting up a hardware based network
was out of question. I had to choose a software simulation which could
realistically model my network and wireless settings and then I could estimate
(conservatively) whether the wireless network could scale to those numbers. We
ended up putting a physical hardware based network three years after the
software simulation and thus software simulation proved a vital step in making
sure that we complete the work in time and with the right expectations.

This article will cover various aspects of simulation and finally why I had to
come up with a framework of my own.

## Choosing the right platform
There are various simulation platforms available such as NS3, Omnet++, Cooja, and Opnet
which can be used for the purpose.
Selection criteria can be:
- Realistic wireless simulation
- Desired MAC layer support
- Scale
- Support for required stacks (in my case, i needed IPv6 based stack)
- How easy is it to plugin a real-world network stack?

In my case, I was experimenting with Contiki and RIOT network stacks and had to
evaluate the performance differences and stack maturity. For mesh network
establishment, RPL protocol was used and I had verify whether the protocol can
actually scale to my requirements.

### Realistic Simulation
Realistic wireless simulation was important because the performance factors
greatly depended on realistic RF. Modules such as
- path-loss modeling,
- asymmetric links,
- probablistic losses,
- interference
had to be simulated and played a key role in deriving the right performance
factors (such as throughput and latencies).


