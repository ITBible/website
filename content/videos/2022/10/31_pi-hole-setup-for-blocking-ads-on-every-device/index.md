---
title: "Pi-Hole Setup for Blocking Ads on Every Device"
description: "Ever wanted a way to block ads across all devices in your network? Why not try Pi-Hole."
summary: "Ever wanted a way to block ads across all devices in your network? Why not try Pi-Hole."
category: Networking
tags: [Tutorial, PiHole]
date: 2022-10-31
slug: pi-hole-setup-for-blocking-ads-on-every-device
authors:
    - "wizardtux"
---
{{< youtubeLite id="wFPqfMegdzc" label="Pi-Hole Setup for Blocking Ads on Every Device" >}}

Official Docs: https://docs.pi-hole.net/main/basic-install/

# Method 1 (Used in video):
```bash
curl -sSL https://install.pi-hole.net | sudo bash
```
# Method 2:
```bash
git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole
cd "Pi-hole/automated install/"
sudo bash basic-install.sh
```

# Method 3:
```bash
wget -O basic-install.sh https://install.pi-hole.net
sudo bash basic-install.sh
```

# Method 4:
Install from docker. See pi-hole documentation for that.

