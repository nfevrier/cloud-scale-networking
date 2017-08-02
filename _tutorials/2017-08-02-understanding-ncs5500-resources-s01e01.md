---
published: true
date: '2017-08-02 19:33 +0200'
title: Understanding NCS5500 Resources (S01E01)
author: Nicolas Fevrier
excerpt: >-
  First post on the NCS5500 Resource: today, we cover the platform and the
  forwarding ASICs
tags:
  - NCS5500
  - NCS 5500
  - LPM
  - LEM
  - eTCAM
position: hidden
---

# Understanding NCS5500 Resources

## S01E01 The Platforms

In the marketing datasheet, you probably read that NCS5501-SE supports up to 2.7M+ routes or that NCS5502 support up to 1.1M routes. It's true, but it's actually a bit more complex since it will not be 2.7M of any kind of routes. So, the logical question will be: how many routes can I actually use ? And one more time, the answer will be: it depends.

This series of posts aim at explaining in details how NCS5500 routers use the different memory resources available for each type of features or prefixes. But we will go further than just discussing "how many routes" and we will try to identify how other type of data (Next-hop, load balancing information, ACL entries, ...) are affecting the scale.

Today, we will start describing the hardware implementation then we will explain how “databases” are used, how are working with profiles, how they can be monitored and troubleshot.

## NCS5500 Portfolio

Routers in the NCS5500 portfolio offer diverse form-factors. Some are fixed (1RU, 2RU), some others are modular (4-slot, 8-slot, 16-slot) with multiple line cards types.

In August 2017, with one exception covered in a followi-up xrdocs post, we are leveraging Qumran-MX or Jericho forwarding ASICs (FA). Qumran is used for System-on-Chip (SoC) routers like NCS5501 and NCS5501-SE, all others systems are using several Jerichos interconnected via Fabric Engines.

We can categorize these systems and line cards in two families:

![Base vs Scale]({{site.baseurl}}/images/base-scale.jpg)


### Using external TCAM (named “Scale" and identified with -SE in the product ID)

- NCS5501-SE

![NCS5501-SE]({{site.baseurl}}/images/5501-se.jpg)

- NCS5502-SE

![NCS5502 ]({{site.baseurl}}/images/5502-.jpg)
  
- NC55-24X100G-SE

![NC55-24X100G-SE]({{site.baseurl}}/images/24x100-se.jpg)

- NC55-24H12F-SE

![NC55-24H12F-SE]({{site.baseurl}}/images/24h12f-se.jpg)

### Not using external TCAM but only the memories inside the FA (named “Base")

- NCS5501

![NCS5501]({{site.baseurl}}/images/5501.jpg)

- NCS5502

![NCS5502 ]({{site.baseurl}}/images/5502-.jpg)

- NC55-36X100G

![NC55-36X100G]({{site.baseurl}}/images/36x100.jpg)

- NC55-18H18F

![NC55-18H18F]({{site.baseurl}}/images/18h18f.jpg)

- NC55-36x100G-S (MACsec card)

![NC55-36X100G-S]({{site.baseurl}}/images/36x100MACsec.jpg)

- NC55-6X200-DWDM-S (Coherent card)

![NC55-6X200G-DWDM-S]({{site.baseurl}}/images/6x200 COH.jpg)


**Note**: Inside a modular chassis, we can mix and match eTCAM and non-eTCAM line cards. A feature is available to decide where the prefixes should be programmed (differentiating IGP and BGP, and using specific ext-communities).
{: .notice--info}

So basically, this external memory used to extend the scale in term of routes and classifiers (Access-lit entries for instance) is what differentiates the systems and line cards. eTCAM should not be confused with the 4GB external packet buffer which is present on the side of each FA, regardless the type of system or line card. The eTCAM only handles prefixes and ACEs, not packets. The external packet buffer will be used in case of queue congestion only. It’s a very rapid graphical memory specifically used for packets.

If you are familiar with traditional IOS XR routers, there are some similarities and some differences with the classification of line cards "-SE vs -TR" on ASR9000, or "-FP vs -MSC vs -LSP" on CRS routers: 
- route and feature scales can be different among the different types of LC
- but not the number of queues or the capability to support Hierarchical QoS (it's not the case for NCS5500 routers, QoS capability is the same on -SE and non-SE)

We have two eTCAM blocks per FA offering up to 2M extra routes in total and they are soldered to the board. It’s not a field-replaceable part, that means you can not convert a NC55-36X100G non-eTCAM card into an eTCAM card.

## Resources / Memories

Each forwarding ASIC is made of two cores (0 and 1). Also we have an ingress and an egress pipeline. 
Each pipeline itself is made of different blocks. 
For clarity and intellectual property reasons, we will simplify the description and represent the series of blocks as just a Packet Processor (PP) and a Traffic Manager (TM).

![Resources]({{site.baseurl}}/images/resources.jpg)

Along the pipeline, the different blocks can access (read or write) different “databases”.
They are memory entities used to store specific type of information.

In follow up posts, we will describe in details how are they used, but let’s introduce them right now.

- The Longest Prefix Match Database (LPM sometimes referred as KAPS for KBP Assisted Prefix Search, KBP being itself Knowledge Based Processor) is an SRAM used to store IPv4 and IPv6 prefixes. It’s an algorithmic memory qualified for 256k entries IPv4 and 128k entries IPv6 in the worst case. We will see it can go much higher with internet distribution.
- The Large Exact Match Database (LEM) is used to store IPv4 and IPv6 routes also, plus MAC addresses and MPLS labels. It scales to 786k entries.
- The Internal TCAM (iTCAM) is used for Packet classification (ACL, QoS) and is 48k entries large.
- The FEC database is used to store NextHop (128k entries), containing also the FEC ECMP (4k entries).
- Egress Encapsulation DB (EEDB) is used for egress rewrites (96k entries), including adjacency encapsulation like link-local details from ARP, ND and for MPLS labels or GRE headers.

All these databases are present inside the Forwarding ASIC.

- The external TCAMs (eTCAM) are only present in the -SE line cards and systems and, as the name implies, are not a resource inside the Forwarding ASIC. They are used to extend unicast route and ACL / classifiers scale (up to 2M IPv4 entries).

Depending on the address family (IPv4 or IPv6), but also depending on the prefix subnet length, routes will be sorted and stored in LEM, LPM or eTCAM. Route handling will dependant on the platform type, the IOS XR released running and the profile activated. That's what we will detail in the next episode.
