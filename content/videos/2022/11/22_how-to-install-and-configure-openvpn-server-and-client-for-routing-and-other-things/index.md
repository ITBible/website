---
title: "How to Install and Configure OpenVPN Server and Client for Routing (and other things)"
description: "There are many things that your own VPN can be used for, from ensuring your ISP isn't spying on you to routing traffic from a data center to your house or even just accessing work resources. Today we're going to talk about installing and configuring a basic OpenVPN setup."
summary: "There are many things that your own VPN can be used for, from ensuring your ISP isn't spying on you to routing traffic from a data center to your house or even just accessing work resources. Today we're going to talk about installing and configuring a basic OpenVPN setup."
categories:
  - Networking
tags: [Tutorial, VPN, OpenVPN, Linux]
date: 2022-11-22
slug: how-to-install-and-configure-openvpn-server-and-client-for-routing-and-other-things
authors:
    - "wizardtux"
---
{{< youtubeLite id="0o2kHw4tU6M" label="How to Install and Configure OpenVPN Server and Client for Routing (and other things)" >}}

I try to keep my posts / tutorials as concise as possible but in some cases I'm not able to do that, and this is one of those cases. This is a longer one but it will be at the core of something I want to do within my lab.

# Install OpenVPN and Download EasyRSA
First run an update and install OpenVPN via apt.
```bash
sudo apt update
sudo apt install openvpn
```
Download and install EasyRSA (in this example we are using version 3.1.1)
```bash
wget -P ~/ https://github.com/OpenVPN/easy-rsa/releases/download/v3.1.1/EasyRSA-3.1.1.tgz
cd ~
tar xvf EasyRSA-3.1.1.tgz
cd ~/EasyRSA-3.1.1
```

# Configure your Certificate Authority
Copy the example vars to a new vars file, it is important to note that this process could be on its own standalone server but for this example we are keeping it all on the same server
```bash
cp vars.example vars
```
edit vars in whatever text editor you want
```bash
nano vars
```
Find the section that sets the defaults for the new certificates
```text
#set_var EASYRSA_REQ_COUNTRY    "US"
#set_var EASYRSA_REQ_PROVINCE   "California"
#set_var EASYRSA_REQ_CITY       "San Francisco"
#set_var EASYRSA_REQ_ORG        "Copyleft Certificate Co"
#set_var EASYRSA_REQ_EMAIL      "me@example.net"
#set_var EASYRSA_REQ_OU         "My Organizational Unit"
```
Un-comment each line and update to your information
```text
set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "Kansas"
set_var EASYRSA_REQ_CITY       "Kansas City"
set_var EASYRSA_REQ_ORG        "IT Bible"
set_var EASYRSA_REQ_EMAIL      "admin@itbible.net"
set_var EASYRSA_REQ_OU         "Community"
```
Initialize the public key infrastructure (pki)
```bash
./easyrsa init-pki
```

# Build the Certificate Authority
for the ease of the tutorial I'm going to use the nopass option so it doesn't prompt for the CA password each time
```bash
./easyrsa build-ca nopass
```
The common name can be any string of characters but for simplicity's sake we're going to use the defaults

# Create the OpenVPN Server's Files Needed for Encryption
Create the private key and certificate request file (aka csr)

we're going to use "pop01.nyc1" in this example but know anywhere I use pop01.nyc1 you can use any name

so lets create our CSR
```bash
./easyrsa gen-req pop01.nyc1 nopass
```
The command we just ran created both pop01.nyc1.req which is the CSR and also created pop01.nyc1.key which is the private key for the certificate.

Now lets copy the private key to our openvpn folder
```bash
sudo cp pki/private/pop01.nyc1.key /etc/openvpn/server.key
```
Now lets sign the request
```bash
./easyrsa sign-req server pop01.nyc1
```
verify the details and to confirm enter yes

if you didn't use nopass in the build-ca step you'll be required to enter a password here

Now we should have an issued certificate in pki/issued/ so lets check.
```bash
ls -la pki/issued/
```
Lets create a Diffie-Hellman key, this will take a few minutes so grab a coffee or something while you wait. This key is used during the key exchange.

I'll fast-forward though
```bash
./easyrsa gen-dh
```
The next step is to create our servers HMAC signature, this helps stregthen the servers TLS integrity verification.
```bash
openvpn --genkey secret ta.key
```
We're going to copy these files into our OpenVPN config for use by the OpenVPN Server
```bash
sudo cp ta.key /etc/openvpn/
sudo cp pki/dh.pem /etc/openvpn/
sudo cp pki/issued/pop01.nyc1.crt /etc/openvpn/server.crt
sudo cp pki/ca.crt /etc/openvpn/ca.crt
```

# Configure Ubuntu for Use with OpenVPN
So there are some aspects of the OS (in this case Ubuntu) that we need to change.

Edit the default IP Forwarding rule
```bash
sudo nano /etc/sysctl.conf
```
remove the comment "#" from the line that contains
```text
...
net.ipv4.ip_forward=1
...
```
then update the current session by running
```bash
sudo sysctl -p
```
Lets find the default interface (for use in a later step)
```bash
ip route | grep default
```
and lets edit the UFW before rules
```bash
sudo nano /etc/ufw/before.rules
```
Just under the start of the file you should have some commented text that looks similar to below, we're going to want to add our next lines right under that.
```text
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#
...
```
so under that description block on the file we're going to want to add our own custom rules that look like below (be sure to leave the lines below it that were there previously, we're just appending between the comment block and the existing rules)
```text
...
# START OPENVPN
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 10.253.0.0/24 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN
...
```
Now you can also do some other really cool stuff in this block, but we'll touch on that in a separate video.

Now we need to tell UFW to allow forwarded packets
```bash
sudo nano /etc/default/ufw
```
find the line ( CTRL+W if you're using nano) DEFAULT_FORWARD_POLICY and change it from DROP to ACCEPT and then save and close the file.

If you didn't change the default port of OpenVPN then we also need to allow that port along with SSH through UFW.
```bash
sudo ufw allow 443/tcp
sudo ufw allow OpenSSH
```
now lets disable and re-enable UFW
```bash
sudo ufw disable
sudo ufw enable
```

# Configure the OpenVPN Service
Copy the sample config over to the OpenVPN folder and then edit it.

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn
sudo nano /etc/openvpn/server.conf
```
For this example we are going to be sure to setup the specific cipher we want as well as convert our port to 443/tcp so that we can get around firewalls that may be blocking based off the port instead of the type of traffic.

You're going to want to use Ctrl+W on this quite a bit and find each of the first parts of the lines, the config below is reflective of what you want it to look like when you're done.
```text
...
port 443 #by default this is 1194
...
proto tcp
...
ca ca.crt
cert server.crt
key server.key
...
dh dh.pem
...
server 10.253.0.0 255.255.255.0 #this can be any private subnet but for my use this is what I chose.
...
tls-auth ta.key 0
...
cipher AES-256-CBC
...
user nobody
group nogroup
...
explicit-exit-notify 0 #this is only required because we are using tcp instead of udp
...
```

# Start and enable OpenVPN
This part is pretty simple if you've ever started and enabled a service in linux, of course if your server config file is not called server.conf then you'll need to change where I use server below.

```bash
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
```

# Check Status
Lets check the status of OpenVPN
```bash
sudo systemctl status openvpn@server
```
it should look something like
```bash
● openvpn@server.service - OpenVPN connection to server
     Loaded: loaded (/lib/systemd/system/openvpn@.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-11-06 05:47:02 UTC; 6 days ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
   Main PID: 5535 (openvpn)
     Status: "Initialization Sequence Completed"
      Tasks: 1 (limit: 2266)
     Memory: 3.4M
        CPU: 9min 12.758s
     CGroup: /system.slice/system-openvpn.slice/openvpn@server.service
             └─5535 /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/server.conf --writepid /run/openvpn/server.pid
```

# Prep for Client Certificates
Lets start off by creating a directory structure for the client configs

```bash
mkdir ~/client-configs/
```
We're going to use the below base configuration file so lets name it base.conf and then edit it.
```bash
nano ~/client-configs/base.conf
```
and paste the below config into it

```text
##############################################
# Sample client-side OpenVPN 2.0 config file #
# for connecting to multi-client server.     #
#                                            #
# This configuration can be used by multiple #
# clients, however each client should have   #
# its own cert and key files.                #
#                                            #
# This configuration was originally modified #
# for use in IT Bible Tutorial find us on    #
# YouTube.com/@ITBible or www.itbible.org    #
#                                            #
# On Windows, you might want to rename this  #
# file so it has a .ovpn extension           #
##############################################
# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client
# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun
# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one.  On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap
# Are we connecting to a TCP or
# UDP server?  Use the same setting as
# on the server.
proto tcp
;proto udp
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote SERVER_IP SERVER_PORT #default port = 1194
# Choose a random host from the remote
# list for load-balancing.  Otherwise
# try hosts in the order specified.
;remote-random
# Keep trying indefinitely to resolve the
# host name of the OpenVPN server.  Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite
# Most clients don't need to bind to
# a specific local port number.
nobind
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nobody
# Try to preserve some state across restarts.
persist-key
persist-tun
# Verify server certificate by checking that the
# certificate has the correct key usage set.
# This is an important precaution to protect against
# a potential attack discussed here:
#  http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the keyUsage set to
#   digitalSignature, keyEncipherment
# and the extendedKeyUsage to
#   serverAuth
# EasyRSA can do this for you.
remote-cert-tls server
# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
# Note that v2.4 client/server will automatically
# negotiate AES-256-GCM in TLS mode.
# See also the data-ciphers option in the manpage
cipher AES-256-CBC
# Set log file verbosity.
verb 3
key-direction 1
; script-security 2
; up /etc/openvpn/update-resolv-conf
; down /etc/openvpn/update-resolv-conf
; script-security 2
; up /etc/openvpn/update-systemd-resolved
; down /etc/openvpn/update-systemd-resolved
; down-pre
; dhcp-option DOMAIN-ROUTE .
```

# Create Client Certificates
Lets create our first client csr you can do this from the client or the server in our case its going to all happen here on the same server
```bash
./easyrsa gen-req home nopass
```
enter your common name (you can just press enter here, but I'm going to name it home.lab.routeto.xyz) and press enter

Lets go ahead and sign it
```bash
./easyrsa sign-req client home
```
Confirm that you're signing a trusted source by entering yes and if you didn't use nopass on the CA you'll need to enter your CA password again.

# Automate the Client Config Creation
So the easiest way to make multiple certificates is to automate the process so everything is uniform. Lets create a script to do all of it for us. So lets create a file called make_config.sh in the ~/client-configs directory.
```bash
nano ~/client-configs/make_config.sh
```
and paste the below script in it

```bash
#!/bin/bash
##
# Original Author Mark Drake
# https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04#step-9-generating-client-configurations
# Slight modifications by WizardTux
# https://itbible.org
##
# First Argument is the common name of the certificate
EASYRSA_DIR=~/EasyRSA-3.1.1
OUTPUT_DIR=~/client-configs
BASE_CONFIG=~/client-configs/base.conf
cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${EASYRSA_DIR}/pki/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${EASYRSA_DIR}/pki/issued/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${EASYRSA_DIR}/pki/private/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${EASYRSA_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```
We'll need this file executable
```bash
chmod +x ~/client-configs/make_config.sh
```

# Create Client Configuration
So since we created our certificate as home, we can use that to create our configuration with the script we just created.
```bash
~/client-configs/make_config.sh home
```
this should create a file for us in ~/client-configs/ called home.ovpn and this contains all of the information needed to connect via your OpenVPN client. You can either cat it out like I'm going to do or you can us something like scp / sftp to copy the file to your computer or wherever you need it to be.

# Login from Windows
In the video I logged into the vpn with a windows client and completed a ping test to the vpn server (my gateway).