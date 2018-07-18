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

<p align="center">
<img src="/images/topology1.png" alt="Sample Topology"/>
</p>

It uses seq=1 and in the future if it has a new state it can use seq=2
such that the peers will know that the new state is the latest state. 

> Now the bigger question here is if the Node A has to restart (or abruptly
> reboots) then what is the sequence number that it can use such that any new
> state that it disemminates is taken as a fresh information from the peers. 

Lets say the Node A reinitializes the sequence number to 1, then other nodes
would consider any new information from node A as stale information.

There are two easy solutions that comes to mind:
1.  Node A could backup the sequence number in persistent storage and on reboot,
    it can restore and increment the sequence number.
2.  Node A could timestamp the packets along with the sequence number or rather
    use timestamp itself as the sequence number.

Problem with approach 1 is the dependence on persistent storage. While it is an
easy fix, persistent storage might not be available or its use might be costly.
For e.g. consider an IoT use-case where the only persistent storage available
is flash and writing to flash is considered costly since the writes to flash
are limited in numbers before the flash sectors go bad.

Problem with approach 2 is that the time across all peers may not be
synchronized.

## Lollipop Counters

Using lollipop counters, the sequence numbers starts with a negative value and
monotonically increments until it reaches zero and then cycles through the
positive set alone.

<p align="center">
<img src="/images/lollipop.png" alt="Lollipop Counter operation"/>
</p>

Lollipop counter has two zones, namely the linear part and the circular part.
The nodes starts in the linear part and moves into circular zone. Once it moves
into the circular part then it stays in the circular part until the next
reboot.

Lollipop counters allows:
1. Peer nodes to detect whether the sender has rebooted
2. Limited use of persistent storage (only in the linear part)
3. the counter sizes to be smaller (because of its circular nature)

### Implications on the use of persistent storage
Use of lollipop counters results in reduced use of flash/persistent storage.
> The sequence number needs to be backed up in persistent storage only for the
> linear part. 
If the node reboots while in circular part, then it restarts in the begining of
the linear part thus the peers knows that the node has restarted and that the
new packets will be the latest pkts.

### Lollipop Counter implementation

Assuming that the sequence window is 16, and the sequence counter size is 1byte.

```
#define SEQ_WIN                 16
#define LPOP_INIT               -(SEQ_WIN)
#define LPOP_INCR(X)            X=(X)<0?++(X):++(X)&0x7f;
#define LPOP_IS_GREATER(A, B)   ((A)>(B) || ((B)-(A))>=SEQ_WIN)
```
[Github: Lollipop Counters](https://github.com/nyrahul/src/tree/master/lollipop)

