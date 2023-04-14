---
title: "Backup"
date: 2023-04-06T22:54:01+02:00
---

## Installation

Remember to install rsync, also on the remote server:

```bash
apt install rsync # on Debian
emerge -av net-misc/rsync # on Gentoo
pacman -S rsync # on Arch
```

## Script for local backups

```bash
#!/bin/bash

srcFolder=$1
destFolder=$2
excludeFileList=${3:-/dev/null}
timestamp=`date +%Y%m%d_%H%M%S`
optionsFirstRun="-avpL --exclude-from=$excludeFileList"
optionsNextRun="$optionsFirstRun --delete --delete-excluded --link-dest=../last"

if [ -d "$destFolder/last" ]; then
  rsync $optionsNextRun $srcFolder/. $destFolder/$timestamp
  rm $destFolder/last && ln -s $timestamp $destFolder/last
else
  rsync $optionsFirstRun $srcFolder/. $destFolder/$timestamp
  ln -s $timestamp $destFolder/last
fi
```

Manuell Usage, where

* srcFolder is the **absolute** path of the source folder,
* destFolder is the **absolute** path of the destination folder (path of the backup) and
* excludeFileList (optional) is file that defines all folders/files to exclude from backup:

```bash
./backup.sh srcFolder destFolder [excludeFileList]
```

Usage in cron (for a hourly backup), where <path>/backup.sh contains the script above:

```bash
# m h  dom mon dow   command
00  *  * * *       <path>/backup.sh srcFolder destFolder [excludeFileList]
```

## Script for remote backups (local2remote)

```bash
#!/bin/bash

srcFolder=$1
server=$2
destFolder=$3
excludeFileList=${4:-/dev/null}
timestamp=`date +%Y%m%d_%H%M%S`
optionsFirstRun="-avpL --exclude-from=$excludeFileList"
optionsNextRun="$optionsFirstRun --delete --delete-excluded --link-dest=../last"

if ssh $server "[ -d '$destFolder/last' ]"; then
  rsync $optionsNextRun $srcFolder/. $server:$destFolder/$timestamp
  ssh $server "rm $destFolder/last && ln -s $timestamp $destFolder/last"
else
  rsync $optionsFirstRun $srcFolder/. $server:$destFolder/$timestamp
  ssh $server "ln -s $timestamp $destFolder/last"
fi
```

Manuell Usage, where

* srcFolder is the **absolute** path of the source folder,
* server is the name of the remote server (you probably want to use ssh-config for this),
* destFolder is the **absolute** path of the destination folder (path of the backup) and
* excludeFileList (optional) is file that defines all folders/files to exclude from backup:

```bash
./backup.sh srcFolder server destFolder [excludeFileList]
```

## Script for remote backups (remote2local)

```bash
#!/bin/bash

server=$1
srcFolder=$2
destFolder=$3
excludeFileList=${4:-/dev/null}
timestamp=`date +%Y%m%d_%H%M%S`
optionsFirstRun="-avpL --exclude-from=$excludeFileList"
optionsNextRun="$optionsFirstRun --delete --delete-excluded --link-dest=../last"

if [ -d "$destFolder/last" ]; then
  rsync $optionsNextRun $server:$srcFolder/. $destFolder/$timestamp
  rm $destFolder/last && ln -s $timestamp $destFolder/last
else
  rsync $optionsFirstRun $server:$srcFolder/. $destFolder/$timestamp
  ln -s $timestamp $destFolder/last
fi
```

Manuell Usage, where

* server is the name of the remote server (you probably want to use ssh-config for this),
* srcFolder is the **absolute** path of the source folder,
* destFolder is the **absolute** path of the destination folder (path of the backup) and
* excludeFileList (optional) is file that defines all folders/files to exclude from backup:

```bash
./backup.sh server srcFolder destFolder [excludeFileList]
```

## Usefull Tricks

### Exclude list for backups of home-directories

A useful exclude list for backups of home-directories is given by

https://github.com/rubo77/rsync-homedir-excludes.

### Check remote server is reachable

You probably only want to run the backup, if the remote server is runnung and reachable.

To do so, add the following code to the backup script:
```bash
ssh -q -o BatchMode=yes -o ConnectTimeout=10 server exit
if [ $? -ne 0 ]; then
    exit 1
fi
```

### Exit Backup Script, if ecrypfts encryption is closed

If you want to use the backup script to backup a via ecrypfts encrypted directory, you probably only want to run the backup, if the encryption is open.

We use the characteristical "Access-Your-Private-Data.desktop" symlink to detect if the encryption is closed.

To do so, add the following code to the backup script:
```bash
if [ -e $srcFolder/Access-Your-Private-Data.desktop ]; then
    exit 2
fi
```