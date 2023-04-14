---
title: "Opendkim"
date: 2023-04-06T22:54:01+02:00
---

This article was initially tested on Debian 9 (stretch) and updated to Debian 10 (buster).

This article is shamelessly copied (and a little modified) from my friend edr, see https://wiki.rochefort.de/sysadmin/mailserver/opendkim.

## Overview

Opendkim is a tool to sign your outgoing mail on a mailserver. The below instructions are largely based on the [debian wiki article for opendkim](/https://wiki.debian.org/opendkim).

## Installation

```bash
apt install opendkim opendkim-tools
```

In Debian 9 (stretch) and Debian 10 (buster) the installed service file did not respect the socket configuration. To fix this, run
```bash
/lib/opendkim/opendkim.service.generate
systemctl daemon-reload
systemctl restart opendkim.service
```

## Create keys

This assumes you already have a postfix config directory.

To create (and configure) a key for a single domain:

```bash
DOMAIN=example.com
mkdir -p /etc/postfix/dkim/
opendkim-genkey -D /etc/postfix/dkim/ -d $DOMAIN -s smtp_$DOMAIN
echo "smtp._domainkey.$DOMAIN $DOMAIN:smtp:/etc/postfix/dkim/smtp_$DOMAIN.private" >> /etc/postfix/dkim/keytable
echo "$DOMAIN smtp._domainkey.$DOMAIN" >> "/etc/postfix/dkim/signingtable"
chgrp opendkim /etc/postfix/dkim/*
chmod g+r /etc/postfix/dkim/*
chown opendkim:opendkim /etc/postfix/dkim/smtp_*.private
chmod 600 /etc/postfix/dkim/smtp_*.private
```

## Configure opendkim

Tell opendkim about the key and signing tables in /etc/opendkim.conf:

```
# Specify the list of keys
KeyTable file:/etc/postfix/dkim/keytable
# Match keys and domains
SigningTable file:/etc/postfix/dkim/signingtable
```

## Configure postfix

In the file /etc/default/opendkim, specify daemon connection settings:

```
SOCKET="inet:8891@localhost"
```

And add the line to the Postfix /etc/postfix/main.cf:

```
milter_default_action = accept
milter_protocol = 2
# As opposed to the debian wiki instructions, do NOT add the below line
#smtpd_milters = inet:localhost:8891
non_smtpd_milters = inet:localhost:8891
```

Add the following line to the submission section in /etc/postfix/master.cf

```
-o smtpd_milters=inet:localhost:8891
```

The consequence of the above modification to instructions you find in other howtos is that you only sign mail that was delivered to you from authenticated clients, rather than e.g. other mailservers trying to impersonate you.

## DNS

Refer to the debian wiki article cited above for instructions on DNS records and testing the setup.
