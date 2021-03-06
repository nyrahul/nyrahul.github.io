---
layout:     post
title:      Art of WSN Simulation for mesh networks
date:       2017-12-8
summary:    WSN simulation is an important aspect for IoT network protocol or
            application development. I ll talk about existing simulation techniques,
            pit-falls, and then introduce Whitefield-framework I developed.
categories: WSN
---

Wireless sensor network simulation plays an important role in development and
deploying wireless networks.
- for systems development, it helps to verify whether the systems assumptions
and protocols will behave in the desired way.
- for networks deployment, it helps to understand whether the networks will
scale to the desired range and whether the desired throughput rate and
latencies could be achieved.

WSN based on low-power RFs such as 802.15.4, aka LoWPAN, is considered for this
article. Having said that, most of the article is generic enough and is
applicable to any wireless technology.

For sceptics, I agree that simulation is not the final answer, but it helps
solve most of the issues before the final deployment. And if issues could be
found in the simulation stage, it saves a lot of cost.

Secondly, not all deployments could be tested with hardware from day one. In
one of my works, the requirement was to setup a 1000 node network with 16 hops
and had to analyze whether performance expectations from this network could
indeed be achieved before the actual development work could be started. Clearly
putting up a hardware based network was out of question. I had to choose a
software simulation which could realistically model my network and wireless
settings and then I could estimate (conservatively) whether the wireless
network could scale to those numbers. We ended up putting a physical hardware
based network two years after the software simulation and thus software
simulation proved a vital step in making sure that we complete the work in time
and with the right expectations.

This article will cover various aspects of simulation and finally why I had to
come up with a framework of my own.

# Choosing the right platform
There are various simulation platforms available such as NS3, Omnet++, Cooja, and Opnet
which can be used for the purpose.
Selection criteria can be:
- Realistic wireless simulation: Can the framework realistics simulate the
  wireless characteristics like path loss, asymmetric links, probabilistics
  losses and collisions/interferences?
- Desired MAC layer support: What kind of MAC layer is supported? Do the MAC
  layer has support for radio duty cycling? Is the required MAC framing support
  present with desired security settings?
- Scale: Can the framework scale to thousands of nodes?
- Using a real-world network stack: Is it possible to experiment on a
  real-world network protocol stack while using the realistic wireless
  simulation?
- Network visualization: Is there enough visualization support such that i can
  check the network topology and other parameters at run time?
- Support for hardware emulation: Does the framework allow actual target
  binaries which could be used on actual hardware to be used in the simulation
  framework? This way we can experiment with actual RAM/ROM requirements of the
  hardware while using the flexibility of the software framework.
- Energy Modelling: Does the framework support measuring energy usage of the
  simulated nodes? How accurate/realistics these measurements are?

In my case, I was experimenting with [Contiki][CONT] and [RIOT][RIOT] network
stacks and had to evaluate the performance differences and stack maturity. For
mesh network establishment, RPL protocol was used and I had to verify whether
the routing protocol can actually scale on 802.15.4 mesh network. Subsequently
I had to verify the best data-rates and per-hop latencies that could be
achieved on such network and what could be the possible bottlenecks.

#### Cooja
There were lot of IEEE papers for sensor networks which made use of Cooja for
simulation purpose and I was sceptic about the results that were achieved. I
understand that not all the experiments conducted need realistic RF but a lot
who need also didn't bother to analyse this. I was specifically interested in
mesh routing protocol and found that lot of papers made claims which were
simply impossible to realize in realistic settings. Cooja does not provide
realistic RF and they do not have the right modelling tools to simulate the RF
effectively.

Having said that Cooja is effectively coupled with Contiki. It makes integrated
testing with Contiki a breeze. Cooja also has a nice UI which specifically
caters to WSNs and thus it is easy to setup a test network and visualize.
Another advantage of Cooja is it allows you to use the OS holistically i.e. the
Contiki network stack can be directly put to test in Cooja. This seems to be
the primary reason why most of the IEEE paper depended on Cooja.

Contiki 802.15.4 mac layer has support for
[various](https://github.com/contiki-os/contiki/wiki/Radio-duty-cycling)
improvised RDC mechanism. Having said that the RDC in this case is not
implemented as part of Cooja but it is implemented as part of native Contiki.

Cooja provides emulation support for famous hardwares such as based on MSP430
platform and for various TI chipsets. Support of emulation allows you to test
with the actual harware binaries on the host linux machine itself. Thus you can
test whether the resource-constrained binary with limited memory can work for
your desired effect.

Other problem with Cooja is its scalability. Cooja uses JVM for simulation and
cannot scale to several 100s of nodes (even if used in batch mode i.e. without
the UI).

Note that Cooja, differs from other simulators in a way that it provides only
physical layer simulation for packets. The whole point about this discussion is
that, Cooja may not be the right tool if your experiment depends on realistic
wireless simulation.

#### NS3


#### Omnet++/Castalia

#### Why I decided to work on Whitefield-Framework?

[CONT]: http://www.contiki-os.org/
[RIOT]: https://www.riot-os.org/
