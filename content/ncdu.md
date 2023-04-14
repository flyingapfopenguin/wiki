---
title: "NCurses Disk Usage (ncdu)"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install ncdu # on Debian
emerge -av sys-fs/ncdu # on Gentoo
pacman -S ncdu # on Arch
```

## Usage

To analyse the disk usage [without crossing filesystem boundaries]

```bash
ncdu [-x] <path>
```