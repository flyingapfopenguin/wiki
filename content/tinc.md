---
title: "Tinc"
date: 2023-04-06T22:54:01+02:00
---

## Overview

Projectpage: https://www.tinc-vpn.org/.

The debian commands in this article were tested on Debian 9 (stretch).

## Installation

First install tinc:
```bash
apt install tinc # on Debian
emerge -av net-vpn/tinc # on Gentoo
pacman -S tinc # on Arch
```

## Configuration

```bash
mkdir -p /etc/tinc/<vpnName>/hosts
```

Create `/etc/tinc/<vpnName>/tinc.conf`:
```
Name = <thisNode>
ConnectTo = <AnotherReachableNode> # optional
#LocalDiscovery = yes
```

Create `/etc/tinc/<vpnName>/tinc-up`:
```
#!/bin/sh
ip link set $INTERFACE up
ip addr add <VPN-IP of this node>/24 dev $INTERFACE
```
and make it executable
```bash
chmod 755 /etc/tinc/<vpnName>/tinc-up
```

Next you need to generate a keypair for this node:
```bash
tincd -n <vpnName> -K4096 # on Debian/Arch (with tinc < 1.1)
tinc -n <vpnName> generate-rsa-keys # on Gentoo
tinc -n <vpnName> generate-ed25519-keys # prospective on Gentoo (with tinc >= 1.1)
```
The private key should be placed in `/etc/tinc/<vpnName>/` with access limited to the root user, e.g.
```bash
chmod 600 /etc/tinc/<vpnName>/rsa_key.priv
```

while the public key should be stored in `/etc/tinc/<vpnName>/hosts/<thisNode>` with the following config:
```
Address = <publicAddressOfThisNode> # only needed, if other nodes should connect to this node
Port = <port> # only needed, if other nodes should connect to this node and not the standart port is used
Subnet = <VPN-IP of this node>/32 # or 0.0.0.0/0 if this node is a exit node for a fulltunnel

-----BEGIN RSA PUBLIC KEY-----
...
-----END RSA PUBLIC KEY-----
```

Concluding you need to exchange the public keys (with the public config) between all hosts that should be able to connect directly.

## Start/Stop VPN

Assume tinc.service is running.

This VPN is controlled via `tinc@<vpnName>.service`.

## Setup full tunnel

To setup a full tunnel, we first configure the firewall on the exit node (server) to forward the traffic, see [Firewall](/firewall), and allow ipv4 forwarding in `/etc/sysctl.conf`:

```
net.ipv4.ip_forward=1
```

On the client we configure a systemd service, with a up and down script:

```bash
#!/bin/sh                                                                                                                                                                                                                                                                                                                    
# /etc/fulltunnel/fulltunnel-up
# see https://www.tinc-vpn.org/examples/redirect-gateway/
VPN_GATEWAY=<VPN-IP of the exit node>
ORIGINAL_GATEWAY=`ip route show | grep default | cut -d ' ' -f 2-5`
REMOTEADDRESS=<public IP of the exit node>
INTERFACE=<vpnName>

ip route add $REMOTEADDRESS $ORIGINAL_GATEWAY
ip route add $VPN_GATEWAY dev $INTERFACE
ip route add 0.0.0.0/1 via $VPN_GATEWAY dev $INTERFACE
ip route add 128.0.0.0/1 via $VPN_GATEWAY dev $INTERFACE
```

```bash
#!/bin/sh                                                                                                                                                                                                                                                                                                                    
# /etc/fulltunnel/fulltunnel-down
# see https://www.tinc-vpn.org/examples/redirect-gateway/
VPN_GATEWAY=<VPN-IP of the exit node>
ORIGINAL_GATEWAY=`ip route show | grep default | cut -d ' ' -f 2-5`
REMOTEADDRESS=<public IP of the exit node>
INTERFACE=<vpnName>

ip route del $REMOTEADDRESS $ORIGINAL_GATEWAY
ip route del $VPN_GATEWAY dev $INTERFACE
ip route del 0.0.0.0/1 via $VPN_GATEWAY
ip route del 128.0.0.0/1 via $VPN_GATEWAY
```

```bash
chmod 755 /etc/fulltunnel/fulltunnel-up
chmod 755 /etc/fulltunnel/fulltunnel-down
```

```
# /etc/systemd/system/fulltunnel.service
[Unit]
Description=Full Tunnel via Tinc VPN
Requires=tinc@<vpnName>.service
After=tinc@<vpnName>.service

[Service]
Type=oneshot
RemainAfterExit=yes
StandardOutput=journal
ExecStartPre=/bin/sleep 2
ExecStart=/etc/fulltunnel/fulltunnel-up
ExecReload=/etc/fulltunnel/fulltunnel-up
ExecStop=/etc/fulltunnel/fulltunnel-down

[Install]
WantedBy=multi-user.target
```

You probably want to use a public DNS to not leak your requested domains beside the full tunnel, see [Static DNS](/networkmanager/#static-dns-for-all-connections).

## Allow connection to another VPN

To allow a node of one VPN (VPN1) to access all nodes of another VPN (VPN2) with the same server, we first configure the firewall on the server to forward the traffic, see [Firewall](/firewall), and allow ipv4 forwarding in `/etc/sysctl.conf`:

```
net.ipv4.ip_forward=1
```

On the client we have to add a route to the other VPN.
To do so we configure a systemd service, with a up and down script:

```bash
#!/bin/sh                                                                                                                                                                                                                                                                                                                    
# /etc/<VPN2>/<VPN2>-up
IP=<VPN2-IP>
INTERFACE=<VPN1name>

ip route add $IP/24 dev $INTERFACE
```

```bash
#!/bin/sh                                                                                                                                                                                                                                                                                                                    
# /etc/<VPN2>/<VPN2>-down
IP=<VPN2-IP>

ip route del $IP/24
```

```bash
chmod 755 /etc/<VPN2>/<VPN2>-up
chmod 755 /etc/<VPN2>/<VPN2>-down
```

```
# /etc/systemd/system/<VPN2>.service
[Unit]
Description=Forward traffic to <VPN2>
Requires=tinc@<vpn1Name>.service
After=tinc@<vpn1Name>.service

[Service]
Type=oneshot
RemainAfterExit=yes
StandardOutput=journal
ExecStartPre=/bin/sleep 2
ExecStart=/etc/<VPN2>/<VPN2>-up
ExecReload=/etc/<VPN2>/<VPN2>-up
ExecStop=/etc/<VPN2>/<VPN2>-down

[Install]
WantedBy=multi-user.target
```

## Other commands

With tinc version >= 1.1 (currently only on Gentoo) you can get a graph of the current state of the network via
```bash
tinc -n <vpnName> dump graph
```

## Move a device to another VPN

To move a device from VPN1 to VPN2 we usually do the following:

- ```rsync -avp /etc/tinc/<VPN1>/ /etc/tinc/<VPN2>```
- change IP adress in config: tinc-up, nodeFile
- substitude configs for other hosts in the VPN
- on a server: change ip in nodeFile and move to other VPN
- sync this change
- on device: ```systemctl stop tinc@<VPN1>.service && systemctl start tinc@<VPN2>.service```
- on a server: ```systemctl restart tinc@<VPN2>.service tinc@<VPN1>.service```
- test
- on device: ```systemctl disable tinc@<VPN1>.service && systemctl enable tinc@<VPN2>.service```
- on device: ```rm -r /etc/tinc/<VPN1>```
- reboot test
