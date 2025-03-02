---
title: "Direct Knowledge Extension to the Distance Vector Routing Protocol"
abbrev: "TODO - Abbreviation"
category: info

docname: draft-todo-yourname-protocol-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: false
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - count to infinity problem
 - distance vector routing
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: Sebastianmueller22/network-protocol
  latest: https://example.com/LATEST

author:
 -
    fullname: Sebastian Mueller
    organization: Ascon Systems GmbH
    email: muellersebastian@mail.com

normative:

informative:


--- abstract

Naive Distance Vector based Routing protocols like RIP (RFC here) suffer from a phenomena called the "count-to-infinity problem"  in the event of a failure event. This Internet draft extends a naive Distance Vector Routing implementation with two simple flags that allow the network to recover quickly and reliably, with no chance of the count to infinity problem to occurr.


--- middle

# Introduction

The count to infinity problem occurs when there is a routing loop, so two connected paths to the same network node exist and a failure event occurs. The nodes within the loop advertise routes via each other to the failed node. Believing each others advertisements they don't notice the failure and continue to route traffic within the routing loop, until the advertisments reach an "infinity" value, in which case failure is assumed and routing traffic to the failed node stops. 
This infinity value poses a restriction on the possible size of the network, because actual routing costs from one end of the network to the other need to remain below that.

This document introduces two flags to distance vector routing, which prevent this problem from occuring. Therefore the strict limit to the size of the network introduced in RIP to limit the impact of the count to infinity problem is not needed. Likewise, split horizon with poisoned reverse is not necessary. The extension is extremely simple.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Count to infinity problem - CTIP

Distance Vector Routing - DVR

Routing Information Protocol - RIP

# The count to infinity problem



# The extension

Two simple concepts are added to a naive DVR implementation. 

First, we add the convention that if a node doesn't receive a Hello message from one of its immediate neighbors for a certain amount of time, it tries to contact it with an are-you-there message which needs to be acknowledged. If this remains unanswered, the node assumes the neighbor to be down. 
It then sends a triggered update on all of its interfaces with the route to the down neighbor being set to -1, or another, similarly impossible value. This signals the failure event to the rest of the network. All receiving nodes that aren't direct neighbors to the failed node perform the following steps:

- They write the infinity value into their routing table
- They stop routing traffic with the failed node as a target and report it to the sender as unreachable
- They ignore any regular updates that advertise a live route to the failed node
- They likewise send a triggered update with this "infinity flag" to all their neighbors

This "bad news" travels with the full speed of triggered updates through the network since all regular advertisements reporting it to be up are ignored. 

This goes on until a node is reached that has the node in question as part of its direct neighbors. The receiving node then tries to reach the failed node. If it answers, the failure was actually a link failure, not a node failure, or the node recovered in the meantime. The node that discovers this then sends a triggerd update with a "direct-knowledge-bit" set. Receiving nodes of such an update message will:

- Write this message into their routing table if they have an infinity value in there or an update message with higher cost and DKB set, or a regular routing cost (they didn't know about the failure yet)
- Trigger an update message advertising this new route with the direct knowledge bit set to all their neighbors
- They start a timeout and retain the DKB until it expires. Any received infinity values within this time and for this node will be ignored
- They route traffic via the new path

So the "good news" that it was actually a link failure or the node is up again travels just as fast through the network. 

Since in the case of a failure no node will believe an update without direct knowledge of the nodes' continuing or regained liveness, count to infinity cannot occur. The best path to the node is discovered as soon as the remaining neighbors hear about the failure through the triggerd updates and the best remaining path is propagated with the speed of triggered updates as well. 


~~~        
     A-----B
      \   / \
       \ /  |
        C  /    all networks have cost 1, except
        | /     for the direct link from C to D, which
        |/      has cost 10
        D
        |<=== target network
~~~

# Security Considerations

- DoS through rouge node
- Privacy through advertisement
- No authorization by default
- etc. etc. 


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
