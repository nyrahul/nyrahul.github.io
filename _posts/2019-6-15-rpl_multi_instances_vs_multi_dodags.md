---
layout:     post
title:      RPL Multi Instances vs Multi-DODAGs
date:       2019-6-15
summary:    RPL routing protocol has a notion of multiple instances and
            multiple DAGs. It is easy to get confused between the two. The post goes into
            the details of differences with use-cases.
categories: rpl
---

[RPL][1] is a distance vector routing protocol for constrained mesh networks
and it introduces the concept of using [multiple instances][2] and multiple
DODAGs (Destination Oriented Directed Acyclic Graph) within the network. An
[RPL][1] Instance contains one or more DODAG roots. There could be more than
one RPL Instances operating in the network.  Figure 1 in [RFC6550][1] shows a
sample network with multiple instances.  However the spec does not explain the
use-cases of these concepts. This is an attempt to differentiate and explain
the use-cases.

> A Node can be part of multiple RPL Instances at the same time but cannot be
> part of multiple DODAGs within the same Instance.

Primary use-case for multiple Instances is to satisfy different applications
with different needs.

Primary use-case for multiple DODAGs within the same Instance is to Load
balance and provide Root failover.

> A Node needs to maintain routing table on per Instance basis. A node may have
> a different next hop for the destination for different Instances.

> RPI (RPL Packet Information RFC 6553) is used to designate data packets to a
> given RPLInstanceID. If the data packet does not contain any RPI extended
> header then the node is free to choose an RPLInstanceID for that packet
> locally.

![Alt text](/images/structure.png "Instances, DODAGs relation")

## Multiple RPL Instances
Multiple RPL Instances could be used to satisfy different application
requirements. Consider a temperature sensor network which sends periodic
temperature reading every 10 minutes. But if the temperature changes
above/below a certain threshold it needs to send the reading immediately and
with less latency as possible. Two RPL Instances could be made use of in this case,
1. satisfies regular periodic traffic and optimizes paths for network lifetime
   (i.e. skips battery powered nodes)
2. satisfies latency-critical traffic when thresholds are crossed and paths
   optimizes for latency and network lifetime.

![Alt text](/images/rpl_multi.png "Multiple Instances")

In this example, green nodes are battery powered and should not be used for
regular periodic traffic. But these battery operated nodes are okay to be used
in latency-critical packets. Thus RPL needs to form two different types of
networks with the same set of nodes. Two RPL Instances, each with its own
objective function to fulfill path characteristics is what is needed. Note that
a node may serve only as a leaf node in a given instance while serving as a
router node in another instance. In the above example, green nodes are leaf
nodes in Instance 0 and can operate as router in Instance 1.

## Multiple DAGs within an Instance
Now lets assume you decide that you don't want a single point of failure in the
form of BR. Thus you need to put another BR in the network. You want this
additional BR to load balance the network when both BRs are active and take
over the whole network when a single BR is operational. This can be achieved by
having multiple DAGs within the same RPL Instance. Each BR will initiate its
own DAG with DODAGID as its own IPv6 address while using the same RPL
InstanceID. Remember that the objective functions and the prefixes still remain
the same. The intermediate nodes (leaf/routers) need to decide to join to
either of the DAGs in the network. They will join the DAG which has the better
metrics and satisfies the given constraints.

<div style="text-align:center" markdown="1">
![Alt text](/images/rpl_multi_dags.png "Multiple DAGs per for Instance=1")
</div>

In this example, the introduction of another BR resulted in load-balancing of
the nodes across the two BRs for RPL Instance=1. This was achieved by using two
DAGs, one initiated by each BR. It is not necessary for all BRs to support all
instances i.e. we can only have two BRs support Instance=1 while Instance=0 is
managed by only one BR.

However note that certain parameters such as RPLInstanceID, prefixes, objective
function information needs to be synced between the two BRs constituting two
DAGs in the same instance. This synchronization is beyond the scope of RPL and
it is assumed that this synchronization is handled on some existing back
channel between the two BRs.

## To sum up

Note that:
1. Multiple RPL Instances could have subset of nodes in common. This is not
   true with multiple DODAGS within an instance i.e. there can't be common
   nodes in multiple DAGs in the same instance.
2. Objective function(OF), prefixes are associated to an RPL Instance. Multiple
   DAGs in the same instance will share this configuration.

[1]: https://tools.ietf.org/html/rfc6550
[2]: https://tools.ietf.org/html/rfc6550#section-3.1.3
