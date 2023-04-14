---
title: "ssh"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install openssh-client openssh-server # on Debian
emerge -av net-misc/openssh # on Gentoo
pacman -S openssh # on Arch
```

## Configuration of the ssh server

Edit the configuration file /etc/ssh/sshd_config.

Deactivate login via password

```
PermitRootLogin without-password
PubkeyAuthentication yes
PasswordAuthentication no
```

You also might want to change the port the ssh server is listening

```
Port <port>
```

You might be interested in configuring

```
AllowAgentForwarding yes/no
X11Forwarding yes/no
```

We uncommented the following lines

```
SyslogFacility AUTH
LogLevel INFO

LoginGraceTime 2m

StrictModes yes
MaxAuthTries 6
MaxSessions 10

HostbasedAuthentication no

IgnoreRhosts yes

PrintLastLog yes
TCPKeepAlive yes
```

and commented

```
#UsePAM yes
```


## Manage the ssh server

The ssh server is managed via the sshd.service.

## Generate ssh key pair

To generate a RSA key pair

```bash
ssh-keygen -t rsa -b 4096
```

To generate an eliptic curve key pair

```bash
ssh-keygen -t ed25519
```

Note: There is no need to set the key size, as all ed25519 keys are 256 bits.

## ssh config

~/.ssh/config looks like

```
Host <yourNameForTheHost>
	Hostname <publicIPorDomain>
	User <user>
	Port <port>
	IdentityFile ~/.ssh/<key>
```

## Authorized keys

Authorized keys are listed (one per line) in ~/.ssh/authorized_keys