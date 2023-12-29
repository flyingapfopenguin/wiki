---
title: "SnappyMail Webmailer"
date: 2023-04-06T22:54:01+02:00
---

*Note*: We used Rainloop Webmailer before it got deprecated, see [the Nextcloud App](https://apps.nextcloud.com/apps/rainloop).

This article was originally tested on Debian 9 (stretch) and later updated to Debian 12 (bookworm).

## Installation and Configuration

Follow the [installation instructions](https://github.com/the-djmaze/snappymail/wiki/Installation-instructions) and see the [release page](https://github.com/the-djmaze/snappymail/releases).

Especially [secure the data folder](https://github.com/the-djmaze/snappymail/wiki/Installation-instructions#secure-data-folder).

Configure Apache suitable, see [Apache Configuration for SnappyMail](/apache#snappymail-webmailer).

Furthermore we usually do the following configuration:
* General
    * Language: German
* Domains
    * add and active your domain
* Login
    * Default Domain: example.com

## Enable Contacts

*Note*: This parted was only tested with Rainloop Webmailer.

This part assumes you have a database like MariaDB, see [MariaDB](/mariadb).

Furthermore the PHP Data Objects (PDO) interface is needed to enable access from PHP to MySQL databases:
```bash
apt install php*-mysql
```

For enabling contacts now follow https://www.howtoforge.com/installation-and-configuration-of-rainloop-in-debian-7-wheezy#-enabling-contacts.
