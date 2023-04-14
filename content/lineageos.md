---
title: "LineageOS"
date: 2023-04-06T22:54:01+02:00
---

This article describes my experience building LinageOS 17.1 for the Oneplus 2 on Gentoo Linux.
Based on the [official build article](https://wiki.lineageos.org/devices/oneplus2/build) I'll note the steps that made me struggle.

## Install the build packages

I had to emerge the following packages

```
dev-util/ccache app-arch/lzop media-gfx/pngcrush sys-process/schedtool sys-fs/squashfs-tools dev-util/android-tools sys-libs/ncurses-compat
```

## Extract proprietary blobs

Extracting the proprietary blobs from the old installation didn't work for me.
Instead I used [this GitHub-Repo](https://github.com/TheMuppets/proprietary_vendor_oneplus) by cloning it to `~/android/lineage/device/oneplus` and checking out the branch `lineage-17.1`.

## Start the build

Before starting the build I had to run

```bash
unset JAVAC # see https://bugs.gentoo.org/646956
ln -s /usr/lib/python-exec/python-exec2 ~/bin/python3 # because the script expects a python-exec wrapped executable in /usr/lib/python-exec
```