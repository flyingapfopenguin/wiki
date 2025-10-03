---
title: "Firewall"
date: 2023-04-06T22:54:01+02:00
---

## General Setup

```bash
#!/bin/bash
# /etc/firewall/firewall.conf
#
# Iptables
FW4="/sbin/iptables"
FW6="/sbin/ip6tables"

# delete existing rules
/etc/firewall/firewall.stop

for FW in {$FW4,$FW6}; do
       # Standard rules
       $FW -P INPUT   ACCEPT
       $FW -P FORWARD DROP
       $FW -P OUTPUT  ACCEPT

       # INPUT
       $FW -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
       $FW -A INPUT -i lo -j ACCEPT -m comment --comment "Allow inbound traffic on lo"
       # Optionally allow inbound ssh
       #$FW -A INPUT -p tcp --dport 22 -j ACCEPT -m comment --comment "Allow inbound ssh"
       # Optionally allow inbound VPN
       #$FW -A INPUT -p tcp --dport <port> -j ACCEPT -m comment --comment "Allow inbound <VPN>"
       # Optionally allow tinc local discovery
       #$FW -A INPUT -p tcp --sport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A INPUT -p tcp --dport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A OUTPUT -p tcp --sport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A OUTPUT -p tcp --dport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A INPUT -p udp --sport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A INPUT -p udp --dport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A OUTPUT -p udp --sport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       #$FW -A OUTPUT -p udp --dport <port> -j ACCEPT -m comment --comment "Allow tinc local discovery <VPN>"
       # Optionally allow inbound http(s)
       #$FW -A INPUT -p tcp --dport 80 -j ACCEPT -m comment --comment "Allow inbound http"
       #$FW -A INPUT -p tcp --dport 443 -j ACCEPT -m comment --comment "Allow inbound https"
       # Optionally allow inbound imap(s)/smtp
       #$FW -A INPUT -p tcp --dport 143 -j ACCEPT -m comment --comment "Allow inbound imap"
       #$FW -A INPUT -p tcp --dport 993 -j ACCEPT -m comment --comment "Allow inbound imaps"
       #$FW -A INPUT -p tcp --dport 587 -j ACCEPT -m comment --comment "Allow inbound smtp"
       #$FW -A INPUT -p tcp --dport 25 -j ACCEPT -m comment --comment "Allow inbound smtp"
       # Optionally allow inbound xmpp protocoll
       #$FW -A INPUT -p tcp --dport 5222 -j ACCEPT -m comment --comment "Allow inbound client xmpp"
       #$FW -A INPUT -p tcp --dport 5269 -j ACCEPT -m comment --comment "Allow inbound server xmpp"
       # Optionally allow inbound jitsi
       #$FW -A INPUT -p tcp --dport 4443 -j ACCEPT -m comment --comment "Allow inbound jitsi"
       #$FW -A INPUT -p udp --dport 4443 -j ACCEPT -m comment --comment "Allow inbound jitsi"
       #$FW -A INPUT -p udp --dport 10000:20000 -j ACCEPT -m comment --comment "Allow inbound jitsi"
       # Optionally allow inbound coturn (for Matrix)
       #$FW -A INPUT -p tcp --dport 3478 -j ACCEPT -m comment --comment "Allow inbound coturn"
       #$FW -A INPUT -p udp --dport 3478 -j ACCEPT -m comment --comment "Allow inbound coturn"
       #$FW -A INPUT -p tcp --dport 5349 -j ACCEPT -m comment --comment "Allow inbound coturn"
       #$FW -A INPUT -p udp --dport 5349 -j ACCEPT -m comment --comment "Allow inbound coturn"
       #$FW -A INPUT -p udp --dport 49152:65535 -j ACCEPT -m comment --comment "Allow inbound coturn"
done

       # Optionally reject inbound ping ...
       #$FW4 -A INPUT -p icmp --icmp-type echo-request -j REJECT -m comment --comment "Explict reject inbound ping"
       #$FW6 -A INPUT -p ipv6-icmp --icmpv6-type echo-request -j REJECT -m comment --comment "Explict reject inbound ping"
       # ... or allow inbound ping
       #$FW4 -A INPUT -p icmp --icmp-type echo-request -j ACCEPT -m comment --comment "Allow inbound ping"
       #$FW6 -A INPUT -p ipv6-icmp -j ACCEPT -m comment --comment "Allow inbound ping"
       #$FW6 -A OUTPUT -p ipv6-icmp -j ACCEPT -m comment --comment "Allow inbound ping"

       # Optionally allow full-tunneling for devices in the <VPN>, see https://wiki.ubuntuusers.de/Tinc/
       #$FW4 -A FORWARD -o eth0 -i <VPN> -s <IP of VPN>/24 -m conntrack --ctstate NEW -j ACCEPT
       #$FW4 -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
       #$FW4 -t nat -A POSTROUTING -o eth0 -s <IP of VPN>/24 -j MASQUERADE

       # Optionally allow forwarding for devices in the <VPN1> towards the <VPN2>, see https://wiki.ubuntuusers.de/Tinc/
       #$FW4 -A FORWARD -o <VPN2> -i <VPN1> -s <IP of VPN1>/24 -m conntrack --ctstate NEW -j ACCEPT
       #$FW4 -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
       #$FW4 -t nat -A POSTROUTING -o <VPN2> -s <IP of VPN1>/24 -j MASQUERADE

for FW in {$FW4,$FW6}; do
       # Drop the rest
       $FW -A INPUT -j DROP
done
```

```bash
#!/bin/sh
# /etc/firewall/firewall.stop
# 
# Iptables
FW="/sbin/iptables"
FW6="/sbin/ip6tables"

# delete existing chains & rules
$FW -F
$FW -X
$FW6 -F
$FW6 -X
```

```bash
chmod 755 /etc/firewall/firewall.conf
chmod 755 /etc/firewall/firewall.stop
```

`/etc/systemd/system/firewall.service`:
```
[Unit]
Description=Custom iptables-based firewall
Wants=network.target
Before=network.target

[Service]
Type=oneshot
RemainAfterExit=yes
StandardOutput=journal
ExecStart=/etc/firewall/firewall.conf
ExecReload=/etc/firewall/firewall.conf
ExecStop=/etc/firewall/firewall.stop

[Install]
WantedBy=multi-user.target
```

## NAT single ports to other subnet

To forward/nat single port(s), in this example for samba server 137, 138 (each udp) and 139, 445 (each tcp), from `<network1>` to `<server>` in `<network2>` use

```bash
iptables -A FORWARD -o <interface for network2> -i <interface for network1> -s <IP of network1>/24 -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -I PREROUTING -p tcp -m multiport --dports 139,445 -j DNAT --to-destination <IP of server (in network2)>
iptables -t nat -I PREROUTING -p udp --dport 137:138 -j DNAT --to-destination <IP of server (in network2)>
iptables -t nat -A POSTROUTING -o <interface for network2> -s <IP of network1>/24 -j MASQUERADE
```
