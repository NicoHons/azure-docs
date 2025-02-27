---
title: Disk capabilities of the hosts
description: Hosts used to build or scale clusters come from an isolated pool of hosts.
ms.topic: include
ms.service: azure-vmware
ms.date: 04/23/2021
author: suzizuber
ms.author: v-szuber
---

<!-- Used in plan-private-cloud-deployment.md and concepts-private-cloud-clusters.md -->


Azure VMware Solution clusters are based on hyper-converged, bare-metal infrastructure. The following table shows the RAM, CPU, and disk capacities of the host.

| Host Type | CPU   | RAM (GB)  | vSAN NVMe cache Tier (TB, raw)  | vSAN SSD capacity tier (TB, raw)  |
| :---      | :---: | :---:     | :---:                           | :---:                             |
| AV36     |  dual Intel 18 core 2.3 GHz  |     576      |                3.2               |                15.20               |

Hosts used to build or scale clusters come from an isolated pool of hosts. Those hosts have passed hardware tests and have had all data securely deleted. 