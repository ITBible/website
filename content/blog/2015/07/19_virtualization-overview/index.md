---
title: 'Archive: Virtualization Overview'
description: 'So about two months ago I started virtualizing my servers and to be honest its probably one of the best decisions I have ever made. There is nothing like saving money and space in my server room.'
summary: 'So about two months ago I started virtualizing my servers and to be honest its probably one of the best decisions I have ever made. There is nothing like saving money and space in my server room.'
category: Hypervisor
tags: [Archive]
date: 2015-07-19
slug: archive-virtualization-overview
authors:
    - "wizardtux"
---

{{< alert icon="fire" cardColor="#e63946" iconColor="#1d3557" textColor="#f1faee" >}}
This is a re-post from a previous version of The Computer Crowd.
{{< /alert >}}

So about two months ago I started virtualizing my servers and to be honest its probably one of the best decisions I have ever made. There is nothing like saving money and space in my server room.

# What is virtualization?
So if you are new to the server side of things or just haven’t branched out much you may not know what virtualization is. Virtualization is the process of taking one physical server and splitting it into several other servers. You can easily try this by running Virtualbox (a Type 2 Hypervisor) on your PC and installing a second operating system (I use this on a regular basis for testing and to run a Windows instance).

# What is a hypervisor?
A hypervisor is a piece of software, firmware, or hardware that runs and monitors virtual instances. There are two main types of hypervisors Type 1 and Type 2:

Type 1 hypervisors are known as bare metal hypervisors (e.g. Proxmox VE, XenServer, Vmware, ESX/ESXI, etc.) this type of hypervisor runs directly on the host machine’s hardware.

Type 2 hypervisors are known as hosted hypervisors (e.g. VirtualBox) this type of hypervisor runs as a distinct software layer on top of the host operating system.

# How can hypervisors help me?
The main reason someone would use a hypervisor is to cut down cost. The cost of power that a single server uses isn’t that huge but if you start thinking about 100s or 1,000s of servers that cost is a huge chunk of change when you virtualize you can drastically cut the power consumption, and in turn, the power bill.

There are many other reasons that people choose to virtualize but this was just a little overview, or introduction if you will, to some of the basics on virtualization.

There are tons of resources out there on the different virtualization platforms but if you have any questions we can help
you out here in the forums or IRC.