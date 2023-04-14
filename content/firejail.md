---
title: "Firejail"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install firejail # on Debian
emerge -av sys-apps/firejail # on Gentoo
pacman -S firejail # on Arch
```

## Usage

To run firefox with firejail use

```bash
firejail --seccomp --private /usr/bin/firefox --no-remote
```