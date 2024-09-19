---
title: "VMs with QEMU und libvirt"
date: 2023-04-06T22:54:01+02:00
---

## Use root session of qemu as user

To access the root session of qemu via a virtmanager run as user ensure that the user is in the libvirt group and the socket permissions are set correctly, e.g.

```bash
$ systemctl cat libvirtd.socket
# /usr/lib/systemd/system/libvirtd.socket
[Unit]
Description=libvirt legacy monolithic daemon socket

[Socket]
ListenStream=/run/libvirt/libvirt-sock
Service=libvirtd.service
SocketMode=0600
RemoveOnStop=yes

[Install]
WantedBy=sockets.target
Also=libvirtd-ro.socket
Also=libvirtd-admin.socket

# /etc/systemd/system/libvirtd.socket.d/override.conf
[Socket]
SocketMode=0660
SocketGroup=libvirt
```

## Network access to VMs

To access VMs via network, e.g. for ssh, we have at least two options.
For VMs using the root session of qemu one can use the bridge network device.
For VMs running with the user session a simple port forwarding is a straightforward solution.

### Network bridge
Qemu should create a *virtual bridge device*, e.g. 'virbr0'.
To use this device without any further configuration in the VM configure DHCP and DNS in /etc/systemd/network/virbr0.network:

```
[Match]
Name=virbr0

[Link]
RequiredForOnline=no

[Network]
Address=192.168.122.1/24
DHCPServer=yes

[DHCPServer]
PoolOffset=10
PoolSize=100
EmitDNS=yes
```

### Port forwarding

For VMs running with the user session of qemu the simplest solution is to configure a port forwarding, e.g.
```bash
virsh qemu-monitor-command --hmp <hostname> 'hostfwd_add ::2222-:22'
```

## Video Setup

Concerning the graphic setup, we made good experience with Spice/VGA during installation and Spice/virtio afterwards (after installing qemu-guest-agent).
