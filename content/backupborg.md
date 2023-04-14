---
title: "BorgBackup"
date: 2023-04-06T22:54:01+02:00
---

Projectpage: https://www.borgbackup.org/.

## Installation

Remember to install borg, also on the remote server: 

```bash
apt install borgbackup # on Debian
emerge -av app-backup/borgbackup # on Gentoo
pacman -S borg # on Arch
```

## Script

```bash
#!/bin/bash

# Skriptvorlage BorgBackup
# https://wiki.ubuntuusers.de/BorgBackup/
# https://borgbackup.readthedocs.io/en/stable/

# Hostname und User
machine=$(hostname)
user=$(whoami)

# Hier Pfad zum Sicherungsmedium angeben.
# z.B. zielpfad="/media/peter/HD_Backup"
server="" # TODO
zielpfad="ssh://$server//<path>" # TODO

# Hier Namen des Repositorys angeben.
# z.B. repository="borgbackups"
repository="$machine-$user"
password="" # TODO

# Archive Name
prefix="$machine-$user"
archive="$prefix-{now:%Y%m%d-%H%M%S}"

# Hier eine Liste mit den zu sichernden Verzeichnissen angeben
# z.B. sicherung="/home/peter/Bilder /home/peter/Videos --exclude *.tmp"
# z.B. sicherung="/root /etc /var/lib/portage /usr/src/linux/.config"
# z.B. sicherung="/root /etc /var/lib /var/www /var/vmail"
sicherung="/home/$user" # TODO

# Exclude File
# z.B. exclude="<absolutePath>/exclude"
exclude="" # TODO (Falls nicht benutzt, unten auskommentieren)

# Hier die Art der Verschlüsselung angeben
# z.B. verschluesselung="none"
verschluesselung="repokey"

# Hier die Art der Kompression angeben
# z.B. kompression="none"
kompression="lz4"

# Hier angeben nach welchem Schema alte Archive gelöscht werden sollen.
# Die Vorgabe behält alle Sicherungen des aktuellen Tages. Zusätzlich das aktuellste Archiv der 
# letzten 7 Sicherungstage, der letzten 4 Wochen sowie der letzten 12 Monate.
# z.B. pruning="--keep-within=1d --keep-daily=7 --keep-weekly=4 --keep-monthly=12"
pruning="--keep-within=2d --keep-daily=14 --keep-weekly=13 --keep-monthly=24 --keep-yearly=100"

###################################################################################################

repopfad="$zielpfad"/"$repository"

# Server erreichbar?
ssh -q -o BatchMode=yes -o ConnectTimeout=10 $server exit
if [ $? -ne 0 ]; then
  echo "Server $server nicht erreichbar."
  exit 1
fi

# Ubuntu home verschlüsselt?
#if [ -e $sicherung/Access-Your-Private-Data.desktop ]; then
#  echo "Home-Verzeichniss ist verschlüsselt."
#  exit 3
#fi

# Backup LDAP
#slapcat -v -l ~/backup.ldif

# backup data
SECONDS=0
echo "Start der Sicherung $(date)."

BORG_PASSPHRASE="$password" borg create --compression $kompression --exclude-from $exclude \
                                 --one-file-system -v --stats --progress \
                                 $repopfad::$archive $sicherung

echo "Ende der Sicherung $(date). Dauer: $SECONDS Sekunden"

# prune archives
BORG_PASSPHRASE="$password" borg prune -v --list $repopfad $pruning
```

To backup a home-directory, we use the following exclude file:

```
#/home/<user>/Downloads
#/home/<user>/LinuxImages
#/home/<user>/.local/share/libvirt
/home/<user>/.local/libvirt
/home/<user>/VirtualBox*
/home/<user>/.gvfs
/home/<user>/.local/share/gvfs-metadata
/home/<user>/.Private
/home/<user>/.dbus
/home/<user>/.cache
/home/<user>/.Trash
/home/<user>/.Trash-1*
/home/<user>/.local/share/Trash
/home/<user>/Trash
/home/<user>/.cddb
/home/<user>/.aptitude
/home/<user>/.npm
/home/<user>/.adobe
/home/<user>/.macromedia
/home/<user>/.xsession-errors
/home/<user>/.wayland-errors
/home/<user>/.local/share/RecentDocuments
/home/<user>/.recently-used
/home/<user>/.recently-used.xbel
/home/<user>/recently-used.xbel
/home/<user>/.thumbnails
/home/<user>/.thumb
/home/<user>/Thumbs.db
/home/<user>/.DS_Store
/home/<user>/.localised
/home/<user>/.bash_history
/home/<user>/.CFUserTextEncoding
/home/<user>/.cups
/home/<user>/.Xauthority
/home/<user>/.ICEauthority
/home/<user>/.gksu.lock
/home/<user>/.pulse
/home/<user>/.pulse-cookie
/home/<user>/.esd_auth
/home/<user>/.kde/share/apps/RecentDocuments
/home/<user>/.kde4/share/apps/RecentDocuments
/home/<user>/.kde/share/apps/klipper
/home/<user>/.kde4/share/apps/klipper
/home/<user>/.kde/share/apps/okular/docdata
/home/<user>/.kde/share/apps/gwenview/recentfolders
/home/<user>/.kde4/share/apps/okular/docdata
/home/<user>/.kde4/share/apps/gwenview/recentfolders
/home/<user>/.kde/share/apps/kmess/displaypics
/home/<user>/.kde4/share/apps/kmess/displaypics
/home/<user>/.kde/share/apps/kmess/customemoticons
/home/<user>/.kde4/share/apps/kmess/customemoticons
/home/<user>/.mozilla/firefox/*/Cache
/home/<user>/.mozilla/firefox/*/minidumps
/home/<user>/.mozilla/firefox/*/.parentlock
/home/<user>/.mozilla/firefox/*/urlclassifier3.sqlite
/home/<user>/.mozilla/firefox/*/blocklist.xml
/home/<user>/.mozilla/firefox/*/extensions.sqlite
/home/<user>/.mozilla/firefox/*/extensions.sqlite-journal
/home/<user>/.mozilla/firefox/*/extensions.rdf
/home/<user>/.mozilla/firefox/*/extensions.ini
/home/<user>/.mozilla/firefox/*/extensions.cache
/home/<user>/.mozilla/firefox/*/XUL.mfasl
/home/<user>/.mozilla/firefox/*/XPC.mfasl
/home/<user>/.mozilla/firefox/*/xpti.dat
/home/<user>/.mozilla/firefox/*/compreg.dat
/home/<user>/.mozilla/firefox/*/pluginreg.dat
/home/<user>/.mozilla/seamonkey/*/Cache
/home/<user>/.mozilla/seamonkey/*/minidumps
/home/<user>/.mozilla/seamonkey/*/.parentlock
/home/<user>/.mozilla/seamonkey/*/blocklist.xml
/home/<user>/.mozilla/seamonkey/*/extensions.sqlite
/home/<user>/.mozilla/seamonkey/*/extensions.rdf
/home/<user>/.mozilla/seamonkey/*/extensions.ini
/home/<user>/.mozilla/seamonkey/*/xpti.dat
/home/<user>/.mozilla/seamonkey/*/compreg.dat
/home/<user>/.mozilla/seamonkey/*/pluginreg.dat
/home/<user>/.thunderbird/*/Cache
/home/<user>/.gnupg/rnd
/home/<user>/.gnupg/random_seed
/home/<user>/.gnupg/.#*
/home/<user>/.gnupg/*.lock
/home/<user>/.gnupg/gpg-agent-info-*
/home/<user>/.config/google-chrome/Default/Local Storage
/home/<user>/.config/google-chrome/Default/Session Storage
/home/<user>/.config/google-chrome/Default/Application Cache
/home/<user>/.config/google-chrome/Default/History Index *
/home/<user>/.config/chromium/Default/Local Storage
/home/<user>/.config/chromium/Default/Session Storage
/home/<user>/.config/chromium/Default/Application Cache
/home/<user>/.config/chromium/Default/History Index *
/home/<user>/.pulse/icons
/home/<user>/.guayadeque/cache.db
/home/<user>/.java/deployment/cache
/home/<user>/.icedteaplugin
/home/<user>/.icedtea
/home/<user>/.gnome2/epiphany/favicon_cache
/home/<user>/.config/libreoffice/4/cache
/home/<user>/.config/freshwrapper-data/Shockwave Flash/WritableRoot/#SharedObjects
/home/<user>/.ccache/?
/home/<user>/.ccache/tmp
/home/<user>/.config/VirtualBox/VBoxSVC.log*
```

## Init

With the config from the script:

```
borg init --encryption=$verschluesselung $repopfad
```

## Usefull commands

To list existing backups:
```bash
borg list $repopfad
```

To mount a backup:
```bash
borg mount $repopfad <mountPoint>
borg umount <mountPoint>
```

## Usage via cron

Usage in cron (for a hourly backup), where <path>/backup.sh contains the script above:

```bash
# m h  dom mon dow   command
00  *  * * *       <path>/backup.sh >> <path>/backup.log 2>&1
```