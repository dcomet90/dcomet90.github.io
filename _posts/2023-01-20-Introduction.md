---
layout: post
title: "Introduction"
date: 2023-01-20 10:00:00 +0200
categories: [homelab,intro]
tags: [homelab,intro]
---


# Welcome

Hello and welcome to my homelab docs site!

All of us in our daily work need to test services or even a specific product or function.
It's not always possible to have a bare metal to do all of the above, so you need a system that will give you what you want without breaking the bank. It's the moment you say to your brain "Let's virtualize everything"

Until recently, at our homelab, the method was a VM machine probably for the everyday user through what we call hypervisor type-2, i.e. a virtual machine installed on an already active operating system, e.g. windows or linux, VMware station or Virtual Box. But if you could in its best form a type 1 hypervisor ie a pure one running directly on bare metal, which is not depend on host hardaware resources.
Then came to establish a new method that simplifies the operation of virtualize and is lighter, containers - either docker or kubernetes.

But the Type 1 hypervisor solution remained an expensive solution, especially if you can't afford to work in a a company that deals with virtualization, let alone have such a system at home. The solution comes with ProxMox Virtual Environment (PVE).

In the following articles, the Hypervisor infrastructure of my Homelab and the first tools where they were installed, such as network monitoirng system (Check MK) and DNS / Ad - Blocker (Pi-Hole) are presented

## Homelab is coming

* Proxmox 7.2.3
* Check MK Community Version
* Pi-Hole


