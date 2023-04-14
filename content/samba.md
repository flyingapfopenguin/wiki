---
title: "Samba"
date: 2023-04-06T22:54:01+02:00
---

## Installation of the samba client

```bash
apt install smbclient # on Debian
emerge -av net-fs/samba # on Gentoo, with USE="client"
pacman -S smbclient # on Arch
```

## List available shares

```bash
smbclient -mNT1 [-U <user>] -L //<IP/hostname>
```

## Mount/Umount share

```bash
mount -t cifs //<IP/hostname>/<ShareName> <MountPoint> -o [vers=<ver>,]username=<username>,password=<password>,uid=<uid>,gid=<gid>,workgroup=<workgroup>,ip=<ip>,iocharset=utf8
umount //<IP/hostname>/<ShareName>
```