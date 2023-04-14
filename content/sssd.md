---
title: "SSSD"
date: 2023-04-06T22:54:01+02:00
---

## Installation
```bash
apt install sssd
```

To configure login run
```bash
pam-auth-update
```
The selected options are stored in /etc/nsswitch.conf.

If you've enabled automatic homedir creation, you might want to set umask for those in `/etc/pam.d/common-session`, e.g.
```
session optional pam_mkhomedir.so [umask=0077]
```

## Configuration

SSSD itself is configured in `/etc/sssd/sssd.conf` (mode 600).
Here is an example configuration (based on the config in https://kifarunix.com/configure-sssd-for-openldap-client-authentication-on-debian-10-9/):
```
[sssd]
services = nss, pam
config_file_version = 2
domains = default
#debug_level = 10 

#[nss]
#override_shell = /bin/bash

[pam]
offline_credentials_expiration = 30

[domain/default]
#ldap_id_use_start_tls = true
cache_credentials = true
account_cache_expiration = 50
ldap_search_base = dc=<domain>,dc=de
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldap://<LDAP URL>
ldap_tls_reqcert = hard
ldap_tls_cacert = /etc/ssl/openldap/certs/cacert.pem
ldap_tls_cacertdir = /etc/ssl/openldap/certs
ldap_search_timeout = 50 
ldap_network_timeout = 60 
access_provider = simple
```

To enable SSL/TLS (see [SSL/TLS for LDAP](/LDAP#Setup SSL/TLS)) copy the above referenced cacert from the LDAP server.

To empty the sssd cache run
```bash
systemctl stop sssd
rm -rf /var/lib/sss/db/*
systemctl restart sssd
```