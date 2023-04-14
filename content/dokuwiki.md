---
title: "DokuWiki"
date: 2023-04-06T22:54:01+02:00
---

## Overview

Projectpage: https://www.dokuwiki.org/dokuwiki.

This article was tested on Debian 9 (stretch).

## Preparation

We assume to have a Webserver, like [Apache](/apache), up an running.
You probably want have a running SSL-Setup, like [Let's Encrypt](/letsencrypt), aswell.

## Installation

First install the dependencies:
```bash
apt install php7.0 libapache2-mod-php7.0 php-xml imagemagick
```

Then download DokuWiki from https://download.dokuwiki.org/ and unpack the archive to your webspace.

Note: You probably don't want to install all languages but may want to install the "Upgrade Plugin".
We only choose the german language and the "Upgrade Plugin".

The official installation guide wants you to read the security page: https://www.dokuwiki.org/security.

Ensure that apache can edit the extracted folder, e.g. by
```bash
chown -R www-data:www-data <path>/dokuwiki
```

Run the web-installer by open install.php in your Webbrowser.

We edited the following settings:

* Language: de
* Wiki Name
* Enable ACL and set Superuser, Realname, E-Mail and Password

For our purpose we choose "Public Wiki" and don't allow users to register themselves.

## Configuration

We configured some settings in the "admin" interface.

### Extension Manager

* "Add New Page" Plugin
* "Markdowku" Plugin (for markdown language)

### Configuration Manager

* "lang": de
* "breadcrumbs": 3
* "dformat: "%d.%m.%Y %H:%M"
* "showuseras": Vollständiger Name des Benutzers
* "sneaky_index": On
* "rememberme": Off
* "mailfrom": dokuwiki@domain.com
* "mailreturnpath": dokuwiki@domain.com
* "htmlmail": Off