---
title: "Nextcloud"
date: 2023-04-06T22:54:01+02:00
---

## Installation

follow https://help.nextcloud.com/t/complete-nc-installation-on-debian-9-stretch-and-manual-update/21881

## LDAP Configuration

see https://docs.nextcloud.com/server/15/admin_manual/configuration_user/user_auth_ldap.html

## Security & setup warnings

Usually we get a few "security & setup warnings", see "Settings" -> "Overview". Here are some notes to work on those:

* PHP memory limit: edit the "memory_limit" option in `/etc/php/<current php version>/apache2/php.ini`
* activate memory cache via APCu, see documentation
* the command to add missing indices to the database is meant to be run as php with the user www-data: "sudo -u www-data php occ db:add-missing-indices"

It might be a good idea to run the security check from Nextcloud.

## Setup a TURN-Server (coturn) for Nextcloud Talk

see https://decatec.de/home-server/nextcloud-talk-mit-eigenem-turn-server-coturn/

## Update process

To check for updates (for apps or the NC instance) use
```bash
sudo -u www-data php occ update:check
```

If updates for apps are available, we can install all of these by
```bash
sudo -u www-data php occ app:update --all
```

One the other hand we can update the NC instance by
```bash
sudo -u www-data php --define apc.enable_cli=1 updater/updater.phar
```

To check your setup use
```bash
sudo -u www-data php occ setupchecks
```
