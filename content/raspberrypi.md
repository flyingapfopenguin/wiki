---
title: "Raspberry Pi"
date: 2023-11-19T22:52:19+01:00
---

# Raspberry Pi OS

## Headless configuration of Raspberry Pi OS

see [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/computers/configuration.html#remote-access).

To enable [ssh](/ssh) add an empty file `boot.txt` to the boot partition.

To create a user add a file `userconf.txt` to the boot partition containing
```
<username>:<encrypted password>
```

You can generate the encrypted password via
```bash
openssl passwd -6
```
