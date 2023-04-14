---
title: "Redshift"
date: 2023-04-06T22:54:01+02:00
---

see https://wiki.archlinux.org/index.php/redshift

## Installation

```bash
apt install redshift # on Debian
emerge -av x11-misc/redshift # on Gentoo
pacman -S redshift # on Arch
```

## Run redshift manuelly

```bash
redshift -l 50.78:6.11 # LAT:LON
```

## Autostart redshift with i3

Uncomment the corresponding line in ~/.i3/config, see [i3](/i3).

