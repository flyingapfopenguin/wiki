---
title: "slim"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install slim # on Debian
emerge -av x11-misc/slim x11-themes/slim-themes # on Gentoo
pacman -S slim slim-themes archlinux-themes-slim # on Arch
```

## Configuration

Slim should use ~./xinitrc by default.

Themes can be found in /usr/share/slim/themes/ and coufigured by the "current_theme" variable in /etc/slim.conf.
On Arch i like "archlinux-simplyblack" theme.

For other options see https://wiki.archlinux.org/index.php/SLiM.

## Manage slim

Slim is managed via the slim.service.
