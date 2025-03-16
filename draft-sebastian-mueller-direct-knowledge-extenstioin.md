---
title: "Direct Knowledge Extension to Distance Vector Routing"
abbrev: ""
category: exp

docname: draft-direct-knowledge-extension
submissiontype: independent  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: false
v: 3
area: AREA
keyword:
 - count to infinity problem
 - distance vector routing
venue:
  mail: muellersebastian@mail.com
  github: Sebastianmueller22/network-protocol

author:
 -
    fullname: Sebastian Mueller
    organization: Ascon Systems GmbH
    email: muellersebastian@mail.com
normative:
  RFC 1058:
    -: ta
    target: https://www.rfc-editor.org/info/rfc1058
    title: Routing Information Protocol
    author:
      name: Hedrick, C.
    date: June 1988
    seriesinfo: DOI 10.17487/RFC1058
    format:
      TXT: https://www.rfc-editor.org/rfc/rfc1058.txt
      PDF: https://www.rfc-editor.org/rfc/pdfrfc/rfc1058.txt.pdf


   RFC 2453:
    -: ta
    target: https://datatracker.ietf.org/doc/rfc2453/
    title: RIP Version 2
    author:
      name: Gary S. Malkin
    date: November 1998
    seriesinfo: DOI 10.17487/RFC2453
    format:
      TXT: https://www.rfc-editor.org/rfc/rfc2453.txt
      PDF: https://www.rfc-editor.org/rfc/pdfrfc/rfc2453.txt.pdf


   RFC 6126
   -: ta
    target: https://www.rfc-editor.org/info/rfc6126
    title: The Babel Routing Protoco
    author:
      name: Juliusz Chroboczek
    date: apr 2011
    seriesinfo: DOI 10.17487/RFC6126
    format:
      TXT: https://www.rfc-editor.org/rfc/rfc6126.txt
      PDF: https://www.rfc-editor.org/rfc/pdfrfc/rfc6126.txt.pdf
   
      

informative:


--- abstract

Naive Distance Vector based Routing protocols like RIP ([RFC 1058](https://www.rfc-editor.org/info/rfc1058)) suffer from a phenomena called the "count-to-infinity problem"  in the event of a network topology change. This Internet draft extends a naive Distance Vector Routing implementation with a simple flag that allows the network to recover quickly and reliably, with no chance of routing loops to occurr.


--- middle

# Introduction

The count to infinity problem arises in distance vector routing protocols when a routing loop forms after a network topology change. In such scenarios, nodes within the loop continue to advertise routes to a failed node through each other. Misled by these advertisements, the nodes fail to recognize the network failure and continue routing traffic within the loop, incrementing the routing metrics until they reach an "infinity" value. At this point, the nodes assume a failure and cease routing traffic to the failed node. The infinity value imposes a limitation on the maximum network size, as the actual routing costs between distant nodes must remain below this threshold.

This document introduces a simple flag to distance vector routing protocols, addressing the count to infinity problem and eliminating the need for strict network size limits imposed by, for example, the Routing Information Protocol (RIP). Consequently, mechanisms such as split horizon with poisoned reverse and feasibility conditions become redundant. The proposed extension is designed to be compatible with "naive" Bellman-Ford based routing protocols like RIP2 ([RFC 2453](https://datatracker.ietf.org/doc/rfc2453/)), rather than more sophisticated protocols like Babel ([RFC 6126](https://www.rfc-editor.org/info/rfc6126)) , which were often themselves developed to tackle the count to infinity problem. Due to its simplicity, this extension should still be compatible with various distance vector-based routing protocols though.


# Conventions and Definitions

Count to infinity problem - CTIP

Distance Vector Routing - DVR

Routing Information Protocol - RIP

# Distance Vector Routing

Explaining the details of Bellmand Ford algorithm based routing is beyond the scope of this document, but in this section a quick overview is given. The basic principle is that nodes only exchange routing information with their direct neighbors. The local link costs are added to the advertised routing cost of a given, more distant node and the path with the lowest cost is chosen. Only the next hop to any given node or network needs to be saved by network participants.

Take this network as an example:

~~~
        A
       / \
      /   \
     B-----C
      \   /
       \ /
        D 
       / \
      /   \
     E     F
      \   /
       \ /
        G
~~~

If the cost between E and G is 10, while all other link costs are 1, E advertises a cost of 10 to D, and F advertises a cost of 1. D calculates the total cost to reach G via E as 11 by adding 1 to E's advertised cost. However, since the path to G via F has a lower cost of 2, D selects F as the next hop to G and updates its routing table accordingly. In the next update cycle, D advertises its path to G through itself, with a total cost of 2.

We will not go through the example further, but it hopefully illustrates the basic workings of these routing protocols. For interested readers, the according sections in the RFCs [6126](https://www.rfc-editor.org/info/rfc6126) and [2453](https://datatracker.ietf.org/doc/rfc2453/) are reccommended.

# Count to infinity

Counting to an arbitrary infinity value is an attempt of naive DVR algorithms such as RIP to resolve routing loops after a network topology change. 

~~~
        A
       / \
      /   \
     B-----C
      \   /
       \ /
        D 
       / \
      /   \
     E     F
      \   /
       \ /
        G
~~~

The lnink costs are the same as in the previous example. When the link between F and G fails, a naive implementation of DVR sets the cost between F and G to an "infinity value" — a positive integer larger than the highest legal routing cost. According to the original RIP [RFC](https://www.rfc-editor.org/info/rfc1058), this value is set to 16. Consequently, F propagates the value of 16 to D. Due to the implementation of split horizon, D does not propagate the cost of 2 back to F. Instead, D selects E as the next best hop to G, updating its cost to G to 11.

B and C do not propagate their cost of 3 to D because of split horizon. However, a routing loop still emerges. In the following update cycle, B and C receive D's updated cost of 11 to G. Despite this, they each advertise their previous cost of 3 to each other and to A. This causes B and C to believe they can route traffic to G via one another, without realizing their paths go through D. As a result, both update their routing tables with a cost of 4 and propagate this new cost to A and D in the next cycle.

In subsequent update cycles, the local link cost of 1 is repeatedly added to the cost in the routing loop. It takes a significant number of cycles for this cumulative cost to eventually exceed 12—the only remaining connection to G, at which point the loop resolves and the network converges to this path.

This takes many update cycles and in the case of a node failure, there is no other way out of it other than to count up to an arbitrary infinity value. This value serves as an arbitrary threshold to terminate the process once it becomes excessively large. While effective in halting the loop, reaching this limit is time-consuming and restricts the range of permissable routing costs and thereby the possible size of the network. 

# The extension

This draft adds the convention that if a node doesn't receive an update message from one of its immediate neighbors for a certain amount of time, it tries to contact it with several messages which need to be acknowledged. If these remain unanswered, the node assumes the neighbor to be down. 
It then sends a triggered update to all other neighbors with the route to the down neighbor being set to -1, or another, similarly impossible value. This can also be implemented as a separate flag in the routing information datagram. It cannot be a positive integer since we make no limitation on the network size. This infinity value signals the failure event to the rest of the network. All receiving nodes that aren't direct neighbors to the failed node MUST perform the following steps:

- They write the infinity value into their routing table
- They stop routing traffic with the failed node as a target and report it to the sender as unreachable
- They ignore any regular updates that advertise a live route to the failed node
- They likewise send a triggered update with this infinity value to all their neighbors

This "bad news" travels with the full speed of triggered updates through the network since all regular advertisements reporting it to be up are ignored. 

This goes on, until a node is reached that has the node in question as part of its direct neighbors. The receiving node then tries to reach the failed node. If it answers, the failure was actually a link failure, not a node failure, or the node recovered in the meantime. The node that discovers this then immidately sends a triggered update with a "direct-knowledge-bit" set. This is best implemented as a bit in the update datagram. Receiving nodes of such an update message MUST:

- Write this message into their routing table if they have an infinity value in there or an update message with higher cost and DKB set, or a regular routing cost (they didn't know about the failure yet)
- Trigger an update message advertising this new route with the direct knowledge bit set to all their neighbors
- Start a timeout and retain the DKB until it expires. Any received infinity values within this time and for this node will be ignored
- Route traffic via the new path

So the "good news" that it was actually a link failure or the node is up again travels just as fast through the network. 

Since in the case of a failure no node will believe an update without direct knowledge of the nodes' continuing or regained liveness, count to infinity cannot occur. All old routing information that potentially is no longer feasible is discarded. The best path to the node is discovered as soon as the remaining neighbors hear about the failure through the triggered updates and the best remaining path is propagated with the speed of triggered updates as well. 

# Example

We again take as an example the following network topology:

~~~
        A
       / \
      /   \
     B-----C
      \   /
       \ /
        D 
       / \
      /   \
     E     F
      \   /
       \ /
        G
~~~
In this example, contrary to before, we assume no other protection against routing loops than the proposed extension. So split horizon is not implemented, just to show that the extension is sufficient to avoid routing loops.

When the link between F and G fails, F sends an infinity value to D. D ignores updates reporting G as up from C,B and E and propagates the infinity value to B,C and E. E knows that G is its direct neighbor and upon receiving the infinity value, it checks if G is reachable. Since it is, it advertises a route to G via itself with the direct knowledge bit set. Meanwhile B and C have sent the infinity value to A. 
D receives the advertisment with the DKB set, ignores the infinity values from F, B and C and propagates this new route to them. B and C ignore the infinity value from A and propagate the new routing information with the DKB to A.

Now the network has converged to the new available Path to G.

# Security Considerations

This document only tries to solve the count to infinity problem. 

The inherent security risks of DVR, such as rogue nodes advertising routes that don't exist, or routes for other nodes to themselves etc. apply here as well. 

The two additions made here pose additional security risks. A rogue node could advertise all other nodes as being down, wether they are its neighbor or not. This would effectively halt communication in the network for a short time and put significant strain on the network while all nodes report their neighbors to still be reachable.

This is not easily preventable, but can be mitigated with another convention, where updates originating from a node need to cryptographically signed before sending. The public key infrastructure necessary for that would need to be implemented on a higher network level.  

That way, repeated infinity values from the same node can be ignored for a certain time (which might be advisable anyway, in the context of frequently failing links).


# IANA Considerations

This document has no IANA actions.


--- back
