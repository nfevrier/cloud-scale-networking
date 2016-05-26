---
title: IOS XR Global Config Replace
author: Jose Liste
published: true
permalink: "/tutorials/ios-xr-global-config-replace"
excerpt: An introduction to Global Config Replace feature in IOS XR
date: "2016-05-18 15:12 -0700"
position: hidden
---

>
In IOS XR 5.3.1, a customer favorite feature was implemented - **Global Configuration Replace**  
>
This feature allows convenient manipulation of router configuration
>
*  Ever wanted to easily move configuration from one interface to another?
*  Or, rather wanted to change all repetitions of a given pattern in your router configuration?
>
If so, keep on reading ...
{: .notice--info}

## Introduction

```
RP/0/0/CPU0:PE1#configure
RP/0/0/CPU0:PE1(config)#replace ?
  interface  replace configuration for an interface
  pattern    replace a string pattern in configuration
```

## Interface-based Replace operation

```
RP/0/0/CPU0:PE1(config)#replace interface <ifid_1> with <ifid_2> ?
  dry-run  execute the command without loading the replace config
  <cr>
```

```
RP/0/0/CPU0:iosxrv-1#show run
Thu May 26 05:46:16.760 UTC
Building configuration...
!! IOS XR Configuration 6.1.1.14I
!! Last configuration change at Thu May 26 05:46:11 2016 by cisco
!
hostname iosxrv-1
interface MgmtEth0/0/CPU0/0
 shutdown
!
interface GigabitEthernet0/0/0/0
 description first
 ipv4 address 10.20.30.40 255.255.255.0
!
interface GigabitEthernet0/0/0/1
 shutdown
!
interface GigabitEthernet0/0/0/2
 description second
 ipv4 address 10.20.50.60 255.255.255.0
!
interface GigabitEthernet0/0/0/3
 shutdown
!
router ospf 10
 area 0
  interface GigabitEthernet0/0/0/0
   transmit-delay 5
  !
 !
!
mpls ldp
 interface GigabitEthernet0/0/0/0
  igp sync delay on-session-up 5
 !
!
end
```

```
RP/0/0/CPU0:iosxrv-1(config)#replace interface gigabitEthernet 0/0/0/0 with gigabitEthernet 0/0/0/2 dry-run
no interface GigabitEthernet0/0/0/0
interface GigabitEthernet0/0/0/2
 description first
 ipv4 address 10.20.30.40 255.255.255.0
router ospf 10
 area 0
  no interface GigabitEthernet0/0/0/0
  interface GigabitEthernet0/0/0/2
   transmit-delay 5
mpls ldp
 no interface GigabitEthernet0/0/0/0
 interface GigabitEthernet0/0/0/2
  igp sync delay on-session-up 5
end
end
RP/0/0/CPU0:iosxrv-1(config)#
RP/0/0/CPU0:iosxrv-1(config)#show
Thu May 26 05:48:51.519 UTC
Building configuration...
!! IOS XR Configuration 6.1.1.14I
end

RP/0/0/CPU0:iosxrv-1(config)#replace interface gigabitEthernet 0/0/0/0 with gigabitEthernet 0/0/0/2
Loading.
365 bytes parsed in 1 sec (357)bytes/sec
RP/0/0/CPU0:iosxrv-1(config)#
RP/0/0/CPU0:iosxrv-1(config)#
RP/0/0/CPU0:iosxrv-1(config)#show
Thu May 26 05:49:10.598 UTC
Building configuration...
!! IOS XR Configuration 6.1.1.14I
no interface GigabitEthernet0/0/0/0
interface GigabitEthernet0/0/0/2
 description first
 ipv4 address 10.20.30.40 255.255.255.0
!
router ospf 10
 area 0
  no interface GigabitEthernet0/0/0/0
  interface GigabitEthernet0/0/0/2
   transmit-delay 5
  !
 !
!
mpls ldp
 no interface GigabitEthernet0/0/0/0
 interface GigabitEthernet0/0/0/2
  igp sync delay on-session-up 5
 !
!
end

RP/0/0/CPU0:iosxrv-1(config)#commit
Thu May 26 05:49:21.767 UTC
RP/0/0/CPU0:iosxrv-1(config)#exit
RP/0/0/CPU0:iosxrv-1#show configuration commit changes last 1
Thu May 26 05:49:38.816 UTC
Building configuration...
!! IOS XR Configuration 6.1.1.14I
interface GigabitEthernet0/0/0/0
 no description first
!
no interface GigabitEthernet0/0/0/0
interface GigabitEthernet0/0/0/2
 description first
 ipv4 address 10.20.30.40 255.255.255.0
!
router ospf 10
 area 0
  no interface GigabitEthernet0/0/0/0
  interface GigabitEthernet0/0/0/2
   transmit-delay 5
  !
 !
!
mpls ldp
 no interface GigabitEthernet0/0/0/0
 interface GigabitEthernet0/0/0/2
  igp sync delay on-session-up 5
 !
!
end

```


## Pattern-based Replace operation

```
RP/0/0/CPU0:PE1(config)#replace pattern ?
  regex-string  pattern to be replaced within single quotes
  
RP/0/0/CPU0:PE1(config)#replace pattern 'regex_1' with 'regex_2' ?
  dry-run  execute the command without loading the replace config
  <cr>  
```

## Examples

```
RP/0/0/CPU0:PE1(config)#replace interface gigabitEthernet 0/0/0/0 with gigabitEthernet 0/0/0/1 dry-run
RP/0/0/CPU0:PE1(config)#replace pattern '10\.20\.30\.40' with '100.200.250.225‘
RP/0/0/CPU0:PE1(config)#replace pattern 'GigabitEthernet0/1/0/([0-4])' with 'TenGigE0/3/0/\1'
```

xxx

## Some Caveats and Considerations

*  Replacing interface "X" with "Y" will cause sub-interfaces (e.g. "X.abc", "X.def") to also be replaced   
Example: replace Gig0/0/0/1 with Gig0/0/0/11 would cause sub-interface Gig0/0/0/1.100 to be replaced to Gig0/0/0/11.100

*  For pattern-based replace, the input is considered a regex string; e.g. replace pattern 'x' with 'y'  
So if you are trying to replace 1.2.3.4 remember to escape the '.' as otherwise it would match any char  
Example: replace pattern '1.2.3.4' with '25.26.27.28' will match and replace both 1.2.3.4 and 10203040  
Example: replace pattern '1\.2\.3\.4' with '25.26.27.28' will match only 1.2.3.4 and not 10203040  

*  Renaming class-maps or flex-cli groups itself with replace may not go thru the commit due to classmap interdependecy on policymap etc.

*  Always use replace “dry-run” keyboard in order to validate changes that would be performed by the replace operation




