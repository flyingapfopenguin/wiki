---
title: "Jitsi"
date: 2023-04-06T22:54:01+02:00
---

## Overview

Projectpage: https://jitsi.org.

This article was tested on Debian 10 (buster).

## Installation

Configure DNS and get a certificate for the (sub-)domian, see [Let's Encrypt](/letsencrypt).

Follow https://jitsi.org/downloads/ubuntu-debian-installations-instructions/ to install the jitsi-meet package. For configuration of the videobridge you are ask to enter the domain of this instance, decide to use your "own" certificate and enter the path to this certificate. Note that the configuration also creates a apache configuration.

## Configuration

### Network configuration

For basic configuration look at https://github.com/jitsi/jitsi-meet/blob/master/doc/quick-install.md#advanced-configuration, especially consider to

* set up the Fully Qualified Domain Name, if your videobridge doesn't work,
* configure Jitsi behind a NAT, if this is your setup,
* configure the firewall, see section "Systemd details" and [Firewall](/firewall).

### Secure domain

If you don't want to run a public Jitsi instance, you have to secure your domain, see https://github.com/jitsi/jicofo#secure-domain.

### LDAP configuration

To authenticate your secured instance via LDAP, follow https://github.com/jitsi/jitsi-meet/wiki/LDAP-Authentication, but set `consider_bosh_secure = true` before the `VirtualHost "localhost"`, e.g. directly after `modules_enabled`.

### Further configuration

For further configuration see the main configuration file at `/etc/jitsi/meet/<domain>-config.js`.
