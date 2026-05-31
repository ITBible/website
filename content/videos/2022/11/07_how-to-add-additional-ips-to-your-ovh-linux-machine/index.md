---
title: "How to add additional IPs to your OVH Linux machine"
description: "This is just a simple method for adding fail over IPs to your Linux server at OVH. This works for both Linux VPS's and dedicated servers."
summary: "This is just a simple method for adding fail over IPs to your Linux server at OVH. This works for both Linux VPS's and dedicated servers."
category: Networking
tags: [Tutorial, Linux, Routing]
date: 2022-11-07
slug: how-to-add-additional-ips-to-your-ovh-linux-machine
authors:
    - "wizardtux"
---
{{< youtubeLite id="N_UXGBnD-88" label="How to add additional IPs to your OVH Linux machine" >}}

This is just a simple method for adding fail over IPs to your Linux server at OVH. This works for both Linux VPS's and dedicated servers.

# Disable Network Config
This file won't be crated by default and we'll be creating it during the next step so lets open it in our editor (in my example I'll be using nano).
```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```
we'll want to add the line below:
```text
network: {config: disabled}
```
and then save and close the file (Ctrl+X, Y, Enter)

# Check which interface our main IP is on
We want to figure out which interface we want to edit.

```bash
ip a
```
This is where the steps can start to vary depending on the distribution that you're using.

# Edit netplan in Ubuntu
Edit netplan cloud-init yaml configuration file (again we are using nano)
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```
By default mine looked like:
```text
network:
    version: 2
    ethernets:
        ens3:
            dhcp4: true
            match:
                macaddress: fa:16:3e:d7:cb:28
            mtu: 1500
            set-name: ens3
```
but we want to update it to this, and spacing does matter in these files so I used the space key just to make sure that we lined up correctly.
```text
network:
    version: 2
    ethernets:
        ens3:
            dhcp4: true
            match:
                macaddress: fa:16:3e:d7:cb:28
            mtu: 1500
            set-name: ens3
            addresses:
            - 158.69.114.73/32
            - 158.69.114.74/32
            - 158.69.114.137/32
            - 158.69.5.77/32
```
then save and close the file.

Now we want to test the changes to the netplan configuration (this is important as yaml formatting can cause issues):
```bash
sudo netplan try
```
If you didn't get disconnected you should be good unless you see any other errors and you can either hit enter (I'd ping your IPs just to make sure they are working) or you can apply the netplan after the fact by running the following command.
```bash
sudo netplan apply
```

# Edit Debian's Cloud Init file
Open the file in your text editor (again I'm using nano).
```bash
sudo nano /etc/network/interfaces.d/50-cloud-init
```
add the following lines (using my example from my ubuntu box it would look something like this):
```text
auto ens0:1
iface ens0:1 inet static
address 158.69.114.73
netmask 255.255.255.255

auto ens0:2
iface ens0:2 inet static
address 158.69.114.74
netmask 255.255.255.255

auto ens0:3
iface ens0:3 inet static
address 158.69.114.137
netmask 255.255.255.255

auto ens0:4
iface ens0:4 inet static
address 158.69.5.77
netmask 255.255.255.255
```
then save and close the file and restart networking
```bash
sudo systemctl restart networking
```
You can also do this on [Windows](https://docs.ovh.com/gb/en/publiccloud/network-services/configure-additional-ip/?ref=itbible.org#windows-server-2016_1) or [CenOS / RedHat variants](https://docs.ovh.com/gb/en/publiccloud/network-services/configure-additional-ip/?ref=itbible.org#cpanel-centos-7-red-hat-derivatives_1).