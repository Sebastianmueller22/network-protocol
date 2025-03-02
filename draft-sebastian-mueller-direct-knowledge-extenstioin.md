---
###
# Internet-Draft Markdown Template
#
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
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

Abstract
Naive Distance Vector based Routing protocols like RIP (RFC here) suffer from a phenomena called the "count-to-infinity problem"  in the event of a failure event. This Internet draft extends the RIP with two simple flags that allow the network to recover quickly and reliably, with no chance of the count to infinity problem to occurr. The basic idea is that the network assumes a node failure in case a direct neighbor reports a loss of connection. All traffic to that node is halted and all routing information that still advertises the failed node as up is discarded, until a direct neighbor of the node reports it to be reachable. This direct knowledge is flagged in the new routing information, making clear that the failure event is known, but the advertised route is still available in spite of that. Then this new route is propagated through the network and traffic to the node resumes via the available link. 


--- middle

# Introduction

The count to infinity problem occurs when there is a routing loop, so two connected paths to the same network node exist and a failure event occurs. The nodes within the loop advertise routes via each other to the failed node. Believing each others advertisements they don't notice the failure and continue to route traffic within the routing loop, until the advertisments reach an "infinity" value, in which case failure is assumed and routing traffic to the failed node stops. 
This infinity value poses a restriction on the possible size of the network, because actual routing costs from one end of the network to the other need to remain below that.

This document introduces two flags to distance vector routing, which prevent this problem from occuring. This is, in contrast to for example BABEL, not an effort to avoid routing loops. Two or more connected paths to the same node can exist without a problem. The fully self-organising nature of a naive Distance Vector Routing based approach is maintained. A loop in the network topology is no longer a problem with this addition. Therefore the strict limit to the size of the network introduced in RIP to limit the impact of the count to infinity problem is not needed. Likewise, split horizon with poisoned reverse is not necessary. 


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Count to infinity problem - CTIP

Distance Vector Routing - DVR

Routing Information Protocol - RIP

# The extension

              ┌───┐              
              │ A │              
              └─┬─┘              
                │                
                │                
                │                
                │                
                │                
                │                
              ┌─┴─┐              
  ┌───────────┤ B ├───────────┐  
  │           └───┘           │  
  │                           │  
  │                           │  
  │                           │  
┌─┴─┐                       ┌─┴─┐
│ C ├───────────────────────┤ D │
└─┬─┘                       └─┬─┘
  │                           │  
  │                           │  
  │                           │  
  │            ┌───┐          │  
  └────────────┤ E ├──────────┘  
               └───┘             


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
