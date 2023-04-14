---
title: "Rainloop Webmailer"
date: 2023-04-06T22:54:01+02:00
---

This article was tested on Debian 9 (stretch) and updated to Debian 10 (buster).

## Installation

see http://www.rainloop.net/downloads/ and http://www.rainloop.net/docs/installation/.

Download the latest version of Rainloop via

```bash
wget www.rainloop.net/repository/webmail/rainloop-latest.zip
```

unzip it and fix the permissions

```bash
chmod 777 data
chmod 777 rainloop
chmod 666 index.php
```

Finally protect your data folder by <DocumentRoot>/data/.htaccess:

```
Deny from all
<IfModule mod_autoindex.c>
Options -Indexes
</IfModule>
```

## Configuration

Configure Apache suitable, see [Apache Configuration for Rainloop](/apache#rainloop-webmailer).

Go to https://<domain>/?admin and login with

```
User: admin
Password: 12345
```

Change the default password.

Furthermore we usually do the following configuration:

* General
    * Language: German
* Domains
    * add and active your domain
* Login
    * Default Domain: example.com

Finally we edit the config file (<DocumentRoot>/data/\_data\_/\_default\_/configs/application.ini):

```
allow_admin_panel = Off
view_editor_type = "PlainForced"
enable = On # Enable logging
auth_logging = On
date_from_headers = Off
allow_ctrl_enter_on_compose = Off
imap_use_auth_plain = Off
imap_use_auth_cram_md5 = On
smtp_use_auth_plain = Off
smtp_use_auth_cram_md5 = On
force_https = On
```

## Enable Contacts

This part assumes you have a database like MariaDB, see [MariaDB](/mariadb).

Furthermore the PHP Data Objects (PDO) interface is needed to enable access from PHP to MySQL databases:
```bash
apt install php*-mysql
```

For enabling contacts now follow https://www.howtoforge.com/installation-and-configuration-of-rainloop-in-debian-7-wheezy#-enabling-contacts.
