---
title: "Unattended Upgrade"
date: 2023-04-06T22:54:01+02:00
---

This article was tested on Debian 10 (buster).

Simply follow https://wiki.debian.org/UnattendedUpgrades. While that

- uncomment all four lines in "Unattended-Upgrade::Origins-Pattern", as described in https://vitux.com/how-to-manage-unattended-upgrades-on-debian-10,
- install `bsd-mailx` for mail notification and
- prefere "Automatic call via /etc/apt/apt.conf.d/02periodic".