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
This feature allows for easy manipulation of router configuration
>
*  Ever wanted the ability to easily move configuration from one interface to another?
*  Or, rather wanted to change all repetitions of a given pattern in your router configuration?
>
If so, keep on reading ...
{: .notice--info}

```
RP/0/0/CPU0:PE1#configure
RP/0/0/CPU0:PE1(config)#replace ?
  interface  replace configuration for an interface
  pattern    replace a string pattern in configuration
```

xxx

```
RP/0/0/CPU0:PE1(config)#replace interface <ifid_1> with <ifid_2> ?
  dry-run  execute the command without loading the replace config
  <cr>
```

xxx

```
RP/0/0/CPU0:PE1(config)#replace pattern ?
  regex-string  pattern to be replaced within single quotes
  
RP/0/0/CPU0:PE1(config)#replace pattern 'regex_1' with 'regex_2' ?
  dry-run  execute the command without loading the replace config
  <cr>  
```

```
RP/0/0/CPU0:PE1(config)#replace interface gigabitEthernet 0/0/0/0 with gigabitEthernet 0/0/0/1 dry-run
```

xxx



