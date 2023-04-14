---
title: "Let's Encrypt"
date: 2023-04-06T22:54:01+02:00
---

see: https://certbot.eff.org.

This article was tested on Debian 9 (stretch).

## Installation

```bash
apt install certbot python3-certbot-apache
```

## Create certificates

```bash
certbot --apache certonly --agree-tos -w /etc/letsencrypt --email <email> --expand -d <domainList> # first run
certbot --apache certonly --expand -d <domainList> # expand certificates
```

## Renew certificates

```bash
certbot renew [--dry-run]
```

## List existing certificates

```bash
certbot certificates
```

##  Usage

For the usage with Apache, see [Apache](/apache).