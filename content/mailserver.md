---
title: "Mailserver"
date: 2023-04-06T22:54:01+02:00
---

This article was initially tested on Debian 9 (stretch) and updated to Debian 10 (buster).

This article is shamelessly copied (and a little modified) from my friend edr, see https://wiki.rochefort.de/sysadmin/mailserver/.

## Overview

A mail server setup consists of several software components that work together but must be installed and configured individually. This page is just an overview of the general process that contains references to other pages which document the indiviual components. 

## DNS

You will need (at least) the following DNS records.

A records:

```
smtp.yourdomain.com
imap.yourdomain.com
mail.yourdomain.com (for the webmailer)
```

A reverse DNS record for

```
smtp.yourdomain.com
```

An MX record for every domain that you want to accept mail for.

The subdomain names are arbitrary, you can pick whatever you like. You could also just use a single domain for everything, but it makes sense to split them up to be prepared for the event that you ever want to split services onto several servers. In that case you don't have to reconfigure your mail clients.

## SSL

See [Let's Encrypt](/letsencrypt).

## SPAM Protection

To integrate spam protection you'll be referencing SPAM assassin in the configuration of Postfix and Dovecot, so it makes sense to already install it upfront. See [Spamassassin](/spamassassin).

## Mail Transfer Agent / SMTP Server

See [Postfix](/postfix).

## Mail Delivery Agent / IMAP Server

See [Dovecot](/dovecot).

## Adding Users

To add a new user, do the following:

* Add an entry to the dovecot passwd database: [Configuration - Authentication / Virtual Users](/dovecot#configuration---authentication--virtual-users)
* Create a virtual mail account in postfix: [Creating mail accounts](/postfix#creating-mail-accounts)

## Optional: Mail signing (DKIM)

See [Opendkim](/opendkim).

## Optional: Webmailer

See [SnappyMail Webmailer](/snappymail).
