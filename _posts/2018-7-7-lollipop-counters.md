---
layout:     post
title:      Sequence numbers and the use of Lollipop Counters
date:       2018-7-7
summary:    Lollipop counters are used in network protocols (especially routing
            protocols) for sequence number operations. The blog talks about the
            operation of lollipop counter, its advantages and an implementation
            experience.
categories: protocols
---

Sequence numbers are used in network protocols to decide whether the received
packet is a latest one or an old one. In case of protocols where there is no
session which governs the scope of sequence numbers, it becomes especially
difficult to decide whether the sequence number is latest or an old one. Let me
explain what I mean by session here...

First lets consider the case of TCP protocol, where it has circular sequence
number and the session initiator randomly initializes the sequence number and
monotonically increments from then onwards. This is a very easy arrangement
because the protocol works P2P i.e. the peers handshake and establish a session
before starting the data movement and thus the scope of sequence number is
managed within the session.

Now lets take an example of any adhoc routing protocol, where the routing state
may be disemminated amongs multiple peers. There is no concept of a session
since the messages flow any to any. If any node restarts/reboots, it has to
ensure that the nodes around believe that the routing updates that it is
sending after restart are fresh ones.

Consider the following topology where Node A starts disemminating its routing
state to the network. 

![Alt text](/images/topology1.png "Sample Topology")

It uses a seq=1 and in the future if it has a new state it can use seq=2
such that the peers will know that the new state is the latest state.

