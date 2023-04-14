---
title: "MariaDB"
date: 2023-04-06T22:54:01+02:00
---

geared to: https://www.digitalocean.com/community/tutorials/how-to-use-mysql-or-mariadb-with-your-django-application-on-ubuntu-14-04.

This article was tested on Debian 9 (stretch).

## Installation

```bash
apt install python3-dev mariadb-server libmariadbclient-dev libssl-dev python3-mysqldb
```

## Configure database

see https://www.digitalocean.com/community/tutorials/how-to-use-mysql-or-mariadb-with-your-django-application-on-ubuntu-14-04, "Create a Database and Database User".

## Backup

It might be a good idea to backup your databases, e.g. via cron
```
00  4  * * *     rm /var/opt/mariadb/backups/dbdump_*.sql; mysqldump --all-databases > /var/opt/mariadb/backups/dbdump_$(date +"\%Y\%m\%d_\%H\%M\%S").sql
```

Remember to add `/var/opt/mariadb/backups` to your backup path (and create the folder, if it doesn't exist).