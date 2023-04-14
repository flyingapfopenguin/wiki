---
title: "GitLab"
date: 2023-04-06T22:54:01+02:00
---

## Overview

Projectpage: https://about.gitlab.com.

This article was tested on Debian 9 (stretch).

## Installation

After configuring DNS for the wanted domain, see https://about.gitlab.com/install/#debian.

To proxy with apache see https://ecollectmedia.com/server/webserver/howto-gitlab-hinter-apache2-ssl-https/.

## Configuration

### Public Registration

To disable public registration, see https://computingforgeeks.com/how-to-disable-user-creation-signup-on-gitlab-welcome-page/.

### ssh

To enable ssh, we had to update the password information of the git user:
```bash
passwd -d git
```

### Registry Garbage Collection

To clean up your registry, e.g. every month
```
00  1  1 * *     gitlab-ctl registry-garbage-collect -m > /dev/null 2>&1
```

## Backup

To backup GitLab data add `/var/opt/gitlab/backups` to your backup paths and make a regular backup, e.g. by
```
00  3  * * *     rm /var/opt/gitlab/backups/*gitlab_backup.tar; gitlab-backup create [SKIP=registry] > /dev/null 2>&1
```