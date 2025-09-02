---
title: BGP (II) - Network Migration via Dynamic Route
date: 2024-07-04
categories: [Network]
tags: [BGP, Dynamic Route]
img_path: /assets/img/network/
---

## Today's purpose

After the emulation, now we are going to verify our migrating strategy: Using BGP dynamic route to smoothy substitute MPLS network with SASE network structure. MPLS has been well-known as a WAN option for most enterprise 

![Img](/assets/img/network/migration_topo.png)

It is known that the MPLS service provider's router, or PE route, is running BGP routing protocol with client's edge router, namely CE router, thus our schema is to equipt our SASE socket directedly connect to CE router. By doing this, we're not only aim at have our new network been broadcasted to enterprise local CE router, but also make new-joining network learn the routes which have prviously been used. Why is that so important? Because we'll never know that whethere all the customer office sites has equipted new socket yet, in short, we have to consider three type of enterprise network structure of WAN region: one, CE router connect to both MPLS link and our socket device; second, only connect to MPLS link; third, only connect to new socket instead.

## What's My Approach

So, how can we achieve it? Furthermore, what approach not only satisfy our requirement, bridging multiple site with various network WAN structure, but also can be deployed global-wide which maybe reach dozens of enterprise office?
Here is where dynamic route comes to handy.

![Img](/assets/img/network/BGP_path_selection.jpg)

My mentor, Wei, told me to use `Local Preference` to make Customer's existing CE router to re-prioritize route metric as the best practice. 

> why use Local Preference?

## Problem Description

Then something weird happened... For some routes which shown below has looping path due to our previous BGP [allowas-in setting](../BGPandMetric/). It makes the network traffic constrained inside these bgp router and cannot go out, causing a "black hole".

![Img](/assets/img/network/bgp_looping.png)

> ps. In BGP routing protocol, router tend to exclusively accept iBGP route from others within same AS group. The purpose of this mechanism is to prevent routing loops.

## How to Solve it

So let me recapitulate this topoloy again, we get three indepedent BGP router from different AS group--PE represent ASN 65000; CE represent ASN 65001; CATO socket represent ASN 65002--and these routers exchange BGP route, either routing table itself or others. Therefore, next question to ask is how do we limit certain routing exchange, so that we can have a clear routing table on our CE router.

![Img](/assets/img/network/migration_topo.png)

![Img](/assets/img/network/itsRouteFilter.jpg)

**Route filter**, it's Route filter. Because for MPLS CE router, it recieve routes mainly from two source--PE router(65000) and SASE socket(65002). Hopefully we want a mechanism to exclude a route from ASN 65000 to be broadcast to ASN 65002 and vice versa, and that mechanism would be **Route Filter**. By filtering out A's routing info told by B's router, we aim at containing the information from A's router itself.

![Img](/assets/img/network/LocalPref.png)

Done. We successfuly purge that extra routing path. I'm sure there should have been more solution to this, and you're definitly welcome to give feedback as long as you know any.

![Img](/assets/img/network/no_looping.png)

As the result, we successfuly and smoothly migrate the traffic flow from orthodox MPLS structure to cutting edge SASE structure with the help of BGP dynamic routing protocols. 

![Img](/assets/img/network/traceroute_BeforeAfter.png)


## Backgroud Knowledge
...TBC...