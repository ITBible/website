---
title: "Designing a Client Network with OSPF - Part 1"
description: "Ever wanted to make your client's network dynamic? In this video we configure a client network for use with OSPF."
summary: "Ever wanted to make your client's network dynamic? In this video we configure a client network for use with OSPF."
categories:
  - Networking
tags: [Tutorial, OSPF, Routing]
date: 2022-11-30
slug: designing-a-client-network-with-ospf-part-1
series: ["Designing a Client Network with OSPF"]
showTableOfContents: true
series_order: 1
authors:
    - "wizardtux"
---
{{< youtubeLite id="TZpfF6swgX0" label="Designing a Client Network with OSPF - Part 1" >}}

## Network Assumptions
For this tutorial we are going to assume the following:

- Physical Layout
    - We have 3 buildings
        - Building 1 (1 switch) - building-1a
            - ports 1-10 - staff access
            - ports 11-15 - guest access
            - ports 16-17 - wireless access points
            - ports 18-19 - connect to building-2a
            - ports 20-21 - unused
            - ports 22-23 - connect to building-3a
        - Building 2 (1 switch) - building-2a
            - ports 1-10 - staff access
            - ports 11-15 - guest access
            - ports 16-17 - wireless access points
            - ports 18-19 - connect to building-1a
            - ports 20-21 - connect to building-3a
            - ports 22-24 - unused (currently)
        - Building 3 (1 switch) - building-3a
            - ports 1-10 - staff access
            - ports 11-15 - guest access
            - ports 16-19 - wireless access points
            - ports 20-21 - connect to building-2a
            - ports 22-23 - connect to building-1a
            - port 24 - connect to fw-3
- VLAN Configuration
    - Default (1)
    - STAFF (10)
    - VOICE (20)
    - GUEST (99)
- Subnet Configuration
    - building1
        - STAFF - 10.10.0.0/24
            - Gateway: 10.10.0.1
            - DHCP Range: 10.10.0.50 - 10.10.0.254
        - VOICE - 10.20.0.0/24
            - Gateway: 10.20.0.1
            - DHCP Range: 10.20.0.50 - 10.20.0.254
        - GUEST - 10.99.0.0/24
            - Gateway: 10.99.0.1
            - DHCP Range: 10.99.0.2 - 10.99.0.254
    - building2
        - STAFF - 10.10.20.0/24
            - Gateway: 10.10.20.1
            - DHCP Range: 10.10.20.50 - 10.10.20.254
        - VOICE - 10.20.20.0/24
            - Gateway: 10.20.20.1
            - DHCP Range: 10.20.20.50 - 10.20.20.254
        - GUEST - 10.99.20.0/24
            - Gateway: 10.99.20.1
            - DHCP Range: 10.99.20.2 - 10.99.20.254
    - building3
        - STAFF - 10.10.30.0/24
            - Gateway: 10.10.30.1
            - DHCP Range: 10.10.30.50 - 10.10.30.254
        - VOICE - 10.20.30.0/24
            - Gateway: 10.20.30.1
            - DHCP Range: 10.20.30.50 - 10.20.30.254
        - GUEST - 10.99.30.0/24
            - Gateway: 10.99.30.1
            - DHCP Range: 10.99.30.2 - 10.99.30.254

## Build Physical Network Layout​
Use the configurations above to build the physical layout in GNS3 or EVE-NG. The Watchguard has been configured with 8 ports and each switch has been configured with 25 ports (the max allowed, and takes in account the MGMT interface).

### Configure pfSense (or other firewall)​
This is not an in depth configuration of pfSense, there are plenty of good examples out there.

#### Create Default Network​
In this example we are going to use port 0 as the WAN and it will be DHCP, and then port 1 will be our LAN on subnet
- 10.0.0.254/24

#### Create Address Aliases​
- StaffNetwork with the subnets
- VoiceNetwork with the subnets
- GuestNetwork with the subnets

#### Create Port Aliases​
- 80, 443, 53 (Guest)

#### Hybrid Outbound NAT​
- Add aliases with Outbound Nat rules

#### Firewall​
Allow staff network to access LAN address of pfSense
Configure Firewall rules to allow outbound traffic for the StaffNetwork (all traffic) and GuestNetwork (TCP/UDP of port alias).

#### Install FRR​
Use package manager to install FRR during the install you must create area first, then interface then set your OSPF and Global settings to enabled, also make sure to check that the following are checked.

- Redistribute Default
- Always Redistribute

## Configure the Routers​
Lets start with building 3 because its where the internet is connected

### Building 3
#### Configure VLANs
Lets first start by naming our switch
```text
configure snmp sysName building-3a
```
Now, lets create our VLANs.
```text
create vlan STAFF tag 10 description "STAFF NETWORK"
create vlan VOICE tag 20 description "VOICE NETWORK"
create vlan GUEST tag 99 description "GUEST NETWORK"
```
now lets configure our ports on this switch, lets start by configuring the AP ports
```text
configure vlan 1 add ports 16-19 untagged
configure vlan 10 add ports 16-19 tagged
configure vlan 99 add ports 16-19 tagged
```
now lets configure the rest of the access ports (staff and guest)
```text
configure vlan 10 add ports 1-10 untagged
configure vlan 99 add ports 11-15 untagged
configure vlan 20 add ports 1-19 tagged
```
at this point I would suggest saving by typing save and hitting enter then pressing "Y" and hitting enter again.

#### Configure Default VLAN and start the OSPF config​
Lets add our IP addresses to our management VLAN 1 and enable IP Forwarding.
```text
configure vlan 1 ipaddress 10.0.0.30/24
enable ipforwarding vlan 1
```
Lets configure OSPF on the first switch, this is not an IP address but in my case I'm going to use the router's IP address.
```text
configure ospf routerid 10.0.0.30
configure ospf add vlan Default area 0.0.0.0 link-type broadcast
enable ospf
```
save your configuration again. If you did everything correctly if you use `show ospf neighbor` you should see something similar to below

```text
Neighbor ID     Pri State           ...     Address
10.0.0.254      1   FULL    /BDR            10.0.0.254
...
```
Total number of neighbors: 1 (All neighbors in Full state)
and if you run show iproute you should see something like
```text
Ori     Destination     Gateway     Mtr ...
#o2     Default Route   10.0.0.254  10  ...
#d      10.0.0.0/24     10.0.0.10   1   ...
#o2     192.168.1.0/24  10.0.0.254  20  ...
```
The last entry being my lan that's visible from the pfSense box. However if you power on a VM connected to any switch at this point you still won't have network access and that's because we don't have any DHCP configured. Now lets configure the staff network with DHCP and add it to our OSPF area.
```text
configure vlan STAFF ipaddress 10.10.30.1/24
enable dhcp ports 1-10 vlan STAFF
configure vlan STAFF dhcp-address-range 10.10.30.50 - 10.10.30.254
configure vlan STAFF dhcp-options default-gateway 10.10.30.1
configure vlan STAFF dhcp-options dns-server primary 208.67.222.222
configure vlan STAFF dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 10
configure ospf add STAFF area 0.0.0.0 link-type broadcast
```
You should now have network connection on your desktop vm connected to port 2 on building-3a. Lets go ahead and do the same for the voice and guest networks.
```text
configure vlan VOICE ipaddress 10.20.30.1/24
enable dhcp ports 1-19 vlan VOICE
configure vlan VOICE dhcp-address-range 10.20.30.50 - 10.20.30.254
configure vlan VOICE dhcp-options default-gateway 10.20.30.1
configure vlan VOICE dhcp-options dns-server primary 208.67.222.222
configure vlan VOICE dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 20
confiugre ospf add VOICE area 0.0.0.0 link-type broadcast
configure vlan GUEST ipaddress 10.99.30.1/24
enable dhcp ports 11-15 vlan GUEST
configure vlan GUEST dhcp-address-range 10.99.30.2 - 10.99.30.254
configure vlan GUEST dhcp-options default-gateway 10.99.30.1
configure vlan GUEST dhcp-options dns-server primary 208.67.222.222
configure vlan GUEST dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 99
configure ospf add GUEST area 0.0.0.0 link-type broadcast
```
now would be a good time to save again while you're checking your pfSense box for the routes. Now lets navigate over to building 1 and get it configured

### Building 1​
Again we're going to create our VLANs
```text
configure snmp sysName building-1a
create vlan STAFF tag 10 description "STAFF NETWORK"
create vlan VOICE tag 20 description "VOICE NETWORK"
create vlan GUEST tag 99 description "GUEST NETWORK"
```
now lets configure our ports on this switch, lets start by configuring the AP ports
```text
configure vlan 1 add ports 16-17 untagged
configure vlan 10 add ports 16-17 tagged
configure vlan 99 add ports 16-17 tagged
```
now lets configure the rest of the access ports (staff and guest)
```text
configure vlan 10 add ports 1-10 untagged
configure vlan 99 add ports 11-15 untagged
configure vlan 20 add ports 1-17 tagged
```
and save, now we're going to configure our management IP address and OSPF
```text
configure vlan 1 ip address 10.0.0.10/24
enable ipforwarding vlan 1
configure ospf routerid 10.0.0.10
configure ospf add vlan Default area 0.0.0.0 link-type broadcast
enable ospf
```
and now we configure our VLANs for this building

#### Staff​
```text
configure vlan STAFF ipaddress 10.10.0.1/24
enable dhcp ports 1-10,16-17 vlan STAFF
configure vlan STAFF dhcp-address-range 10.10.0.50 - 10.10.0.254
configure vlan STAFF dhcp-options default-gateway 10.10.0.1
configure vlan STAFF dhcp-options dns-server primary 208.67.222.222
configure vlan STAFF dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 10
configure ospf add STAFF area 0.0.0.0 link-type broadcast
```

#### VOICE​
```text
configure vlan VOICE ipaddress 10.20.0.1/24
enable dhcp ports 1-17 vlan VOICE
configure vlan VOICE dhcp-address-range 10.20.0.50 - 10.20.0.254
configure vlan VOICE dhcp-options default-gateway 10.20.0.1
configure vlan VOICE dhcp-options dns-server primary 208.67.222.222
configure vlan VOICE dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 20
configure ospf add VOICE area 0.0.0.0 link-type broadcast
```

#### GUEST​
```text
configure vlan GUEST ipaddress 10.99.0.1/24
enable dhcp ports 11-17 vlan GUEST
configure vlan GUEST dhcp-address-range 10.99.0.2 - 10.99.0.254
configure vlan GUEST dhcp-options default-gateway 10.99.0.1
configure vlan GUEST dhcp-options dns-server primary 208.67.222.222
configure vlan GUEST dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 99
configure ospf add GUEST area 0.0.0.0 link-type broadcast
```
and save again.

### Building 2​
#### Switch building-2a​
Again we're going to create our VLANs
```text
configure snmp sysName building-2a
create vlan STAFF tag 10 description "STAFF NETWORK"
create vlan VOICE tag 20 description "VOICE NETWORK"
create vlan GUEST tag 99 description "GUEST NETWORK"
```
now lets configure our ports on this switch, lets start by configuring the AP ports
```text
configure vlan 1 add ports 16-17 untagged
configure vlan 10 add ports 16-17 tagged
configure vlan 99 add ports 16-17 tagged
```
now lets configure the rest of the access ports (staff and guest)
```text
configure vlan 10 add ports 1-10 untagged
configure vlan 99 add ports 11-15 untagged
configure vlan 20 add ports 1-17 tagged
```
and save, now we're going to configure our management IP address and OSPF
```text
configure vlan 1 ip address 10.0.0.20/24
enable ipforwarding vlan 1
configure ospf routerid 10.0.0.20
configure ospf add vlan Default area 0.0.0.0 link-type broadcast
enable ospf
```
and now we configure our VLANs for this building

##### Staff
```text
configure vlan STAFF ipaddress 10.10.20.1/24
enable dhcp ports 1-10,16-17 vlan STAFF
configure vlan STAFF dhcp-address-range 10.10.20.50 - 10.10.20.254
configure vlan STAFF dhcp-options default-gateway 10.10.20.1
configure vlan STAFF dhcp-options dns-server primary 208.67.222.222
configure vlan STAFF dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 10
configure ospf add STAFF area 0.0.0.0 link-type broadcast
```

##### VOICE
```text
configure vlan VOICE ipaddress 10.20.20.1/24
enable dhcp ports 1-17 vlan VOICE
configure vlan VOICE dhcp-address-range 10.20.20.50 - 10.20.20.254
configure vlan VOICE dhcp-options default-gateway 10.20.20.1
configure vlan VOICE dhcp-options dns-server primary 208.67.222.222
configure vlan VOICE dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 20
configure ospf add VOICE area 0.0.0.0 link-type broadcast
```

##### GUEST
```text
configure vlan GUEST ipaddress 10.99.20.1/24
enable dhcp ports 11-15 vlan GUEST
configure vlan GUEST dhcp-address-range 10.99.20.2 - 10.99.20.254
configure vlan GUEST dhcp-options default-gateway 10.99.20.1
configure vlan GUEST dhcp-options dns-server primary 208.67.222.222
configure vlan GUEST dhcp-options dns-server secondary 208.67.220.220
enable ipforwarding vlan 99
configure ospf add GUEST area 0.0.0.0 link-type broadcast
```
and save again.

Interested in Part 2? See it here.