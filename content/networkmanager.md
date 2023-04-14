---
title: "NetworkManager"
date: 2023-04-06T22:54:01+02:00
---

## Installation

On Gentoo we use the following global USE-Flags to let NetworkManager manage our network connections.

```
networkmanager
```

```bash
apt install network-manager network-manager-openvpn network-manager-vpnc # on Debian
emerge -av net-misc/networkmanager net-misc/networkmanager-openvpn net-misc/networkmanager-vpnc # on Gentoo
pacman -S networkmanager networkmanager-openvpn networkmanager-vpnc # on Arch
```

## Configuration

### Static DNS for all connections

To set a static DNS server for all connections managed by NetworkManager, create/edit /etc/NetworkManager/NetworkManager.conf to configure NetworkManager to not overwrite DNS settings

```
[main]
dns=none
```

and configure one or more static dns servers in /etc/resolv.conf, e.g.

```
nameserver 1.0.0.1 # Cloudflare, etc., see https://1.0.0.1/
nameserver 1.1.1.1 # Cloudflare, etc., see https://1.1.1.1/
nameserver 9.9.9.9 # IBM, etc. (mailware filtered), see https://www.quad9.net/
nameserver 9.9.9.10 # IBM, etc. (not mailware filtered)
nameserver 74.82.42.42 # Hurricane Electric LLC
#nameserver 8.8.8.8 # Google
#nameserver 8.8.4.4 # Google
```

## CLI

Use nmtui.