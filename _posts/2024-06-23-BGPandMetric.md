---
title: BGP (I) - Emulating MPLS Service
date: 2024-06-25
categories: [Network]
tags: [BGP, Dynamic Route]
img_path: /assets/img/network/
---

## Problem Description

I was working on a project require me to emulate a customer MPLS enviornment, so that we can do furthter networking architecture migration and recontruction.
In brief, below is a simple emulated MPLS backbone network running iBGP with ASN 65000. And CE router also has its own BGP reign with ASN 65001, which means it runs eBGP with the service provider.

```
CE1 <--eBGP--> PE1 <--iBGP--> PE2 <--eBGP--> CE2
```

So here is a problem: when I `show ip bgp` in four machine, I certainly cannot see the BGP routing tables in both CE1/CE2 router, while there exist the BGP routing flows in PE routers.
![Img](Before_Allow.png)

What reason was that PE can recieve the BGP advertising information, which CE can't?

## How to Solve it

My co-worker and also probably the best mentor, Wei, told me the answer lie behind in how BGP is designed to prevent routing loops by rejecting routes that contain the local AS number in the AS path as default behavior. This mechanism helps ensure that a route is not advertised back into the AS from which it originated, which could cause routing loops and instability in the network.

Thus the solution is pretty simple, by adding a function `allowas-in`, the CE routers will then be able to recieve routes of ASN 65001 from PE routers.
 
From:
```
router bgp 65001
 bgp log-neighbor-changes
 network 1.1.1.5 mask 255.255.255.255
 redistribute connected
 neighbor 192.168.1.1 remote-as 65000
```

To:
```
router bgp 65001
 bgp log-neighbor-changes
 network 1.1.1.5 mask 255.255.255.255
 redistribute connected
 neighbor 192.168.1.1 remote-as 65000
 neighbor 192.168.1.1 allowas-in
```

As a result, our CE routers have obtain the BGP routes to the other site.
![Img](After_Allow.png)

## Backgroud Knowledge

Because I've experienced solid four years of computer science education and training process, I always prefered to think dynamic routing protocol in a algorithm way(yes, that algorithm implemeneted when it comes to graph):
1. link-state
    - Algorithm: `Bellman-Ford` algorithm.
    - Route Calculation: Based on the distance to the destination and the direction (vector).
    - Information Sharing: Each router shares its entire routing table with its immediate neighbors at regular intervals.
    - Convergence Time: Generally slower; prone to issues like counting to infinity.
    - Complexity: Simpler to implement and understand.
    - Scalability: Less scalable; suitable for smaller networks.
    - Examples: RIP, IGRP.
2. distance-vector
    - Algorithm: `Dijkstra's` algorithm.
    - Route Calculation: Each router constructs a complete map (link-state database) of the network topology and calculates the shortest path.
    - Information Sharing: Routers exchange link-state advertisements (LSAs) with all other routers in the network.
    - Convergence Time: Faster; more efficient handling of topology changes.
    - Complexity: More complex to implement and requires more resources.
    - Scalability: More scalable; suitable for larger networks.
    - Examples: OSPF, IS-IS.
3. hybrid
    - Algorithm: Combination of distance-vector and link-state techniques.
    - Route Calculation: Uses distance-vector techniques for quick updates and link-state techniques for accurate information.
    Examples: EIGRP.
4. path-vector
    - Algorithm: `Path-vector` algorithm.
    - Route Calculation: Based on the path information (list of ASes) that a route has traversed.
    - Information Sharing: Each router shares the path information with its peers, including the entire AS path.
    - Convergence Time: Depends on path propagation; can be slower in very large networks.
    - Complexity: High flexibility and control over route selection using various attributes.
    - Scalability: Highly scalable; suitable for inter-domain routing.
    - Examples: BGP.

![Img](routing_protocol_sum.png)

And todays's topic aim at explaining BGP: 

### Metric

### Local Preference

### Weight

### Path


