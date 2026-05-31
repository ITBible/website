---
title: 'Archive: Proxmox VE Review'
description: 'So since January I have been virtualizing all of my servers with Proxmox VE and I must say I am very happy with it.'
summary: 'So since January I have been virtualizing all of my servers with Proxmox VE and I must say I am very happy with it.'
category: Hypervisor
tags: [Archive, Linux]
date: 2015-07-19
slug: archive-proxmox-ve-review
authors:
    - "wizardtux"
---

{{< alert icon="fire" cardColor="#e63946" iconColor="#1d3557" textColor="#f1faee" >}}
This is a re-post from a previous version of The Computer Crowd.
{{< /alert >}}

Readers Note: If you have no idea what I am talking about from the title of this article please read [my previous post](/blog/archive-virtualization-overview/) first.

So since January I have been virtualizing all of my servers with Proxmox VE and I must say I am very happy with it.

For those of you who have never used Proxmox VE it uses both KVM and Open VZ. It is considered a Type 1 (or bare metal) hypervisor, though some people may consider it a Type 2 (or hosted) hypervisor because it uses a Linux kernel and is based off of Debian/GNU. Proxmox VE has a lot of features, some that I have yet to use and we won’t cover them all here. Some of the features are clustering (in a multi-master design), web interface, RESTful API, role based
authentication, high availability (HA), bridged networking, and flexible storage options.

The reason I chose to use Proxmox VE over any other open source solution, is because it is managed by a web interface (and is by default) and the reason I wanted that was because I didn’t want to worry about trying to get client software to install on my other machines (mainly because I only run Ubuntu). I have tested clustering though haven’t used it in production and it works well. I also haven’t had the pleasure of using the HA features due to virtualizing a virtualizing platform. One of the only things that I have found that you need the command line for is clustering, though even then its not difficult and there is plenty of help out there.

Proxmox VE has been very stable, very friendly to use and I have only ran into one issue with it and I’m not convinced that its a server side issue. The issue I ran into was when using the console option from the web gui most of the time the console screen is bigger than the VNC window it uses. Editing the GRUB config has made this easier to deal with though sometimes it will revert back to being too big.

If you are looking to start virtualizing in a testing or even production environment I would urge you to take a look at Proxmox VE.

