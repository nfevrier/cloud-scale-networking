---
published: false
date: '2017-08-07 13:46 +0200'
title: Understanding NCS5500 Resources (S01E03)
author: Nicolas Fevrier
excerpt: Third post on the NCS5500 Resources focusing on IPv6 prefixes
position: hidden
---
{% include toc icon="table" title="Understanding NCS5500 Resources" %} 

## S01E03 IPv6 Prefixes

### Previously on "Understanding NCS5500 Resources"

In the previous posts, we introduced the [different routers and line cards in NCS5500 portfolio](https://xrdocs.github.io/cloud-scale-networking/tutorials/2017-08-02-understanding-ncs5500-resources-s01e01/) and we explained [how IPv4 prefixes are sorted in LEM, LPM and eTCAM](https://xrdocs.github.io/cloud-scale-networking/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/).

All the principles described below and the examples used to illustrate them were validated in August 2017 with Jericho-based systems, using scale (with eTCAM) and base (without eTCAM) line cards and running the two IOS XR releases available: 6.1.4 and 6.2.2.
{: .notice--info}

### IPv6 routes and FIB Profiles

Spend a few minutes to read the S01E02 to understand the different databases used to store routes in NCS5500:

![Resources]({{site.baseurl}}/images/resources.jpg){: .align-center}

- **LPM**: Longest Prefix Match Database (or KAPS) is an SRAM used to store IPv4 and IPv6 prefixes. 
- **LEM**: Large Exact Match Database also used to store specific IPv4 and IPv6 routes, plus MAC addresses and MPLS labels.
- **eTCAM**: external TCAMs, only present in the -SE "scale" line cards and systems. As the name implies, they are not a resource inside the Forwarding ASIC, it's an additional memory used to extend unicast route and ACL / classifiers scale.

We explained how the different profiles influenced the prefixes storing in different databases for base and scale systems or line cards. The principles for IPv6 are similar but things are actually simpler: the order of operation will be exactly the same, regardless of the FIB profile activated and regardless of the type of line card (**base** or **scale**).

![IPv6-order.jpg]({{site.baseurl}}/images/IPv6-order.jpg){: .align-center}

The logic behind this decision is that IPv6/48 prefixes are by far the largest population of the public table:

![ipv6-table.jpg]({{site.baseurl}}/images/ipv6-table.jpg){: .align-center}
(From https://twitter.com/bgp6_table){: .align-center}

To avoid any misunderstanding, let's review the IPv6 resource allocation / distribution for each profile and line card type quickly:

![IPv6-base-host-distr.jpg]({{site.baseurl}}/images/IPv6-base-host-distr.jpg){: .align-center}
Base systems with Host-optimized FIB profile{: .align-center}

![IPv6-base-internet-distr.jpg]({{site.baseurl}}/images/IPv6-base-internet-distr.jpg){: .align-center}
Base systems with Internet-optimized FIB profile{: .align-center}

![IPv6-scale-distribution.jpg]({{site.baseurl}}/images/IPv6-scale-distribution.jpg){: .align-center}
Scale systems regardless of FIB profile{: .align-center}

See ? Pretty easy. By default, IPv6/48 are moved into LEM and the all other IPv6 prefixes are pushed into LPM.

### Lab verification

LPM is an algorithmic memory. That means, the capacity will depend on the prefix distribution. We will use a couple of examples below to illustrate below how the routes are moved but you should not rely on the "estimated capacity" to based your capacity planning. Only a real internet table will give you a reliable estimation of the available space.

In slot 0/0, we have a **base line card** (18H18F) using an Internet-optimized profile. In slot 0/6, we use a **scale line card** (24H12F-SE).

Also, keep in mind we are announcing ordered prefixes which are fine in a lab context to verify where the system will store the routes but it's not a realistic scenario (compared to a real internet table for instance).

__IPv6/48 Routes__

IPv6/48 prefixes are stored in LEM:

![IPv6-48.jpg]({{site.baseurl}}/images/IPv6-48.jpg){: .align-center}

First we advertise 20,000 IPv6/48 routes and check the different databases.
On **Base line cards**:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh route ipv6 bgp | i /48 | utility wc -l
<mark>20000</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lem location 0/0/CPU0

HW Resource Information
    Name                            : lem

OOR Information
    NPU-0
        Estimated Max Entries       : 786432
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 20107    (3 %)
        iproute                     : 5        (0 %)
        ip6route                    : <mark>20000</mark>    (3 %)
        mplslabel                   : 102      (0 %)

RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/0/CPU0

HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 117926
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 172      (0 %)
        iproute                     : 29       (0 %)
        ip6route                    : 117      (0 %)
        ipmcroute                   : 50       (0 %)

RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

Note: for readability, we will only display NPU-0 information. In the full output of the show command, we will have from NPU-0 to NPU-0 on 18H18F and from NPU-0 to NPU-3 on the 24H12F-SE.
{: .notice--info}

On **Scale line cards**:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lem location 0/6/CPU0

HW Resource Information
    Name                            : lem

OOR Information
    NPU-0
        Estimated Max Entries       : 786432
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
    NPU-0
        Total In-Use                : 20127    (3 %)
        iproute                     : 24       (0 %)
        ip6route                    : <mark>20000</mark>    (3 %)
        mplslabel                   : 102      (0 %)


RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/6/CPU0

HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 118638
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 144      (0 %)
        iproute                     : 0        (0 %)
        ip6route                    : 117      (0 %)
        ipmcroute                   : 50       (0 %)

RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

With 20,000 IPv6/48 prefixes, as expected, it's only 3% of the 786,432 entries of LEM.

Just for verification, we will advertise 200,000 then 400,000 IPv6/48 routes. And of course the LEM estimated max entries will stay constant. LEM is very different than LPM from this perspective.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh route ipv6 bgp | i /48 | utility wc -l
200000
RP/0/RP0/CPU0:NCS5508-1-614#
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lem location 0/0/CPU0  | i "(Estim|In-Use)"
        Estimated Max Entries       : 786432
        Total In-Use                : <mark>200107   (25 %)</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/0/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 117926
        Total In-Use                : 172      (0 %)
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lem location 0/6/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 786432
        Total In-Use                : <mark>200127   (25 %)</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/6/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 118638
        Total In-Use                : 144      (0 %)
RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh route ipv6 bgp | i /48 | utility wc -l
<mark>400000</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lem location 0/0/CPU0  | i "(Estim|In-Use)"
        Estimated Max Entries       : 786432
        Total In-Use                : <mark>400107   (51 %)</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/0/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 117926
        Total In-Use                : 172      (0 %)
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lem location 0/6/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 786432
        Total In-Use                : <mark>400127   (51 %)</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/6/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 118638
        Total In-Use                : 144      (0 %)
RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

__Non IPv6/48 Routes ?__

From IPv6/1 to IPv6/47 and from IPv6/49 to IPv6/128, all these prefixes will be stored in LPM.

![IPv6-47-0.jpg]({{site.baseurl}}/images/IPv6-47-0.jpg){: .align-center}

![IPv6-128-49.jpg]({{site.baseurl}}/images/IPv6-128-49.jpg){: .align-center}

The estimated max prefixes will be very different for each test and will also differ depending on the number of routes we advertise.

__IPv6/32 Routes__

Let's see is the occupation for 20,000 / 40,000 and 60,000 IPv6/32 prefixes.

On **base line cards**:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh route ipv6 bgp | i /32 | utility wc -l
<mark>20000</mark>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/0/CPU0
HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 493046
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 20172    (4 %)
        iproute                     : 29       (0 %)
        ip6route                    : <mark>20117    (4 %)</mark>
        ipmcroute                   : 50       (0 %)

RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

On **scale line cards**:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/6/CPU0
HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 492362
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 20144    (4 %)
        iproute                     : 0        (0 %)
        ip6route                    : <mark>20117    (4 %)</mark>
        ipmcroute                   : 50       (0 %)

RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

With 40,000 IPv6/48 prefixes:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/0/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 477274
        Total In-Use                : 40172    (8 %)
RP/0/RP0/CPU0:NCS5508-1-614#
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/6/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 478572
        Total In-Use                : 40144    (8 %)
RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

With 60,000 IPv6/48 prefixes:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/0/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 459800
        Total In-Use                : 60172    (13 %)
RP/0/RP0/CPU0:NCS5508-1-614#
RP/0/RP0/CPU0:NCS5508-1-614#sh contr npu resources lpm location 0/6/CPU0 | i "(Estim|In-Use)"
        Estimated Max Entries       : 460686
        Total In-Use                : 60144    (13 %)
RP/0/RP0/CPU0:NCS5508-1-614#
</code>
</pre>
</div>

__IPv6/48 Routes__

__IPv6/64 Routes__

__IPv6/128 Routes__



<div class="highlighter-rouge">
<pre class="highlight">
<code>

</code>
</pre>
</div>



