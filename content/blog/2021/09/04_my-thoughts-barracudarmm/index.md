---
title: 'My thoughts on Barracuda RMM'
description: 'Our experience in using a remote monitoring and management experience and who we switched to after loosing 100s of devices and experiencing slow page loads and lack-luster support.'
summary: 'Our experience in using a remote monitoring and management experience and who we switched to after loosing 100s of devices and experiencing slow page loads and lack-luster support.'
categories:
  - Monitoring & Management
tags: [Networking, Barracuda, Connectwise]
date: 2021-09-04
slug: my-thoughts-on-barracuda-rmm
authors:
    - "wizardtux"
---

I got hired on at this MSP in 2016, and when I was hired my initial job was to setup and get their recently purchased Remote Management and Monitoring System (RMM). This RMM back then was AVG Managed Workplace (it was previously Level Platforms Managed Workplace). Coming from a construction background I thought this system was awesome. Little did I know that my experience was going to get so bad to where the system was basically unusable.

Back then I feel like Managed Workplace was a good system, everything worked as advertised though the requirements were a little rough we basically had to dedicate a Virtual Machine (VM) to its Onsite Manager with 4-8GB of Memory and around 2-4 cores. It was rough but it worked.

Then Avast bought it. I would say it stayed about the same and I don't remember any new "features during this time."

And then it was purchased by Barracuda...

![Old rusted truck](old-rusted-truck.jpg)

Honestly the only things that Barracuda has added is that they have tried to integrate their own products into it (though we never used any of them). Support has gone down hill. Last year (2020) we have 100's of devices disappear from the system all together and at this point I completely lost faith in the system.

Loosing 100's of devices is bad enough, right? It couldn't get any worse, could it? – Yes it could... So starting in June (of 2021) page loads started to take 5,10,15 minutes just to navigate around the system, the remote software just wouldn't work, and you'd randomly get kicked out of remote sessions and our answer from our account team was basically "stop emailing us it will be fixed in the next update" (it wasn't by the way, but it is a little better).

![Blue plastic robot](blue-plastic-robot-toy.jpg)

In comes our savior ConnectWise Automate. I think the biggest thing about Automate that I like is that its more of a framework to build our own RMM rather than something that is pre-built containing a bunch of stuff we don't need. We are currently in the transition of this and I've posted some of my experience in the IT Bible Forums but will have a more extensive write up coming in the future (probably October / November timeframe).

What has your experience been with Managed Workplace or what RMM do you use?