---
title: "vim"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install vim # on Debian
emerge -av app-editors/vim # on Gentoo
pacman -S vim # on Arch
```

## Configuration

~/.vimrc:

```bash
set number
syntax on
set background=dark
set ts=2
set scrolloff=10
set is hls
```