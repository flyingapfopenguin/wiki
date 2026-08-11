---
title: "Dovecot"
date: 2023-04-06T22:54:01+02:00
---

This article was tested on Debian 13 (trixie) and addresses Dovecot 2.4 (and above).
The [original article](https://github.com/flyingapfopenguin/wiki/blob/51b14cd7c3ac291ada8a122ded8160cf5f7f4549/content/dovecot.md) addressed Dovecot 2.3 (and below), was initially tested on Debian 9 (stretch) and updated to Debian 12 (bookworm) and shamelessly copied (and a little modified) from my friend edr, see https://wiki.rochefort.de/sysadmin/mailserver/dovecot.

## Overview

Dovecot gives you some additional features in comparison to [Postfix](/postfix) to deliver incoming mail. It also allows you to access your mail via IMAP.

## Installation

```bash
apt install dovecot-imapd dovecot-sieve
```

## Configuration

The dovecot configuration is split up into many individual files that live in `/etc/dovecot/conf.d`.

### Authentication / Virtual Users

```diff
# diff --color --unified 10-auth.conf.ucf-dist 10-auth.conf
--- 10-auth.conf.ucf-dist
+++ 10-auth.conf
@@ -90,7 +90,7 @@
 #   plain login digest-md5 cram-md5 ntlm anonymous gssapi
 #   gss-spnego xoauth2 oauthbearer
 # NOTE: See also auth_allow_cleartext setting.
-#auth_mechanisms = plain login
+auth_mechanisms = plain cram-md5

 ##
 ## Password and user databases
@@ -103,11 +103,26 @@
 # duplicating the system users into virtual database.
 #
 # <https://doc.dovecot.org/latest/core/config/auth/passdb.html>
+passdb passwd-file {
+  driver = passwd-file
+  passwd_file_path = /etc/dovecot/passwd
+}
+
 #
 # User database specifies where mails are located and what user/group IDs
 # own them. For single-UID configuration use "static" userdb.
 #
 # <https://doc.dovecot.org/latest/core/config/auth/userdb.html>
+userdb static {
+  driver = static
+
+  fields {
+    allow_all_users = yes
+    gid = mail
+    home = /var/vmail/%{user|domain}/%{user|username}
+    uid = vmail
+  }
+}

 #!include auth-deny.conf.ext
 #!include auth-master.conf.ext
```

To define the user / password database, the above config refers to `/etc/dovecot/passwd`.
This file must be defined using the following syntax:

```
user1@example.com:{CRAM-MD5}password1::::::
user2@example.com:{CRAM-MD5}password2::::::
```

The password hashes can be generated via:
```bash
doveadm pw -s CRAM-MD5
```

The path to the home directory contains placeholders, being `%{user|domain}` for the domain and `%{user|username}` for the local part of mailadresses.

```diff
# diff --color --unified auth-system.conf.ext.ucf-dist auth-system.conf.ext        :(
--- auth-system.conf.ext.ucf-dist
+++ auth-system.conf.ext
@@ -10,7 +10,7 @@
 # REMEMBER: You'll need /etc/pam.d/dovecot file created for PAM
 # authentication to actually work. <https://doc.dovecot.org/latest/core/config/auth/databases/pam.html>
 passdb pam {
-#  driver = pam
+   driver = pam
 #  session = yes
 #  setcred = yes
 #  failure_show_msg = yes
@@ -39,10 +39,10 @@
 ## User databases
 ##
 
-# System users (NSS, /etc/passwd, or similiar). In many systems nowadays this
+# System users (NSS, /etc/passwd, or similar). In many systems nowadays this
 # uses Name Service Switch, which is configured in /etc/nsswitch.conf.
-#userdb passwd-file {
-  #driver = passwd-file
+userdb passwd-file {
+  driver = passwd-file
   #auth_username_format=%{user|lower}
   #passwd_file_path = /etc/passwd
   #fields {
@@ -51,7 +51,7 @@
   #  home = /var/vmail/%{user}
   #}
   #skip = found
-#}
+}
 
 # Static settings generated from template <https://doc.dovecot.org/latest/core/config/auth/databases/static.html>
 #userdb static {
```

### Mail User

```diff
# diff --color --unified 10-mail.conf.ucf-dist 10-mail.conf
--- 10-mail.conf.ucf-dist
+++ 10-mail.conf
@@ -34,9 +34,7 @@
 # at
 # https://doc.dovecot.org/2.4.1/core/config/mailbox/formats/mbox.html
-mail_driver = mbox
-mail_home = /home/%{user | username}
+mail_driver = maildir
 mail_path = %{home}/mail
-mail_inbox_path = /var/mail/%{user}

 # If you need to set multiple mailbox locations or want to change default
 # namespace settings, you can do it by defining namespace sections.
@@ -191,15 +189,15 @@
 # to make sure that users can't log in as daemons or other system users.
 # Note that denying root logins is hardcoded to dovecot binary and can't
 # be done even if first_valid_uid is set to 0.
-#first_valid_uid = 500
-#last_valid_uid = 0
+first_valid_uid = 150
+last_valid_uid = 150

 # Valid GID range for users, defaults to non-root/wheel. Users having
 # non-valid GID as primary group ID aren't allowed to log in. If user
 # belongs to supplementary groups with non-valid GIDs, those groups are
 # not set.
-#first_valid_gid = 1
-#last_valid_gid = 0
+first_valid_gid = 8
+last_valid_gid = 8

 # Maximum allowed length for mail keyword name. It's only forced when trying
 # to create new keywords.
```

The user id and group id must be in line with the system account you created in [Creating mail accounts](/postfix#creating-mail-accounts).

### Postfix Authentication via Dovecot

```diff
# diff --color --unified 10-master.conf.ucf-dist 10-master.conf                  :(
--- 10-master.conf.ucf-dist
+++ 10-master.conf
@@ -107,9 +107,11 @@
   }

   # Postfix smtp-auth
-  #unix_listener /var/spool/postfix/private/auth {
-  #  mode = 0666
-  #}
+  unix_listener /var/spool/postfix/private/auth {
+    mode = 0660
+    user = postfix
+    group = postfix
+  }

   # Auth process is run as this user.
   #user = $SET:default_internal_user
```

### SSL

```diff
# diff --color --unified 10-ssl.conf.ucf-dist 10-ssl.conf
--- 10-ssl.conf.ucf-dist
+++ 10-ssl.conf
@@ -3,7 +3,7 @@
 ##

 # SSL/TLS support: yes, no, required. <https://doc.dovecot.org/latest/core/config/ssl.html>
-ssl = yes
+ssl = required

 # PEM encoded X.509 SSL/TLS certificate and private key.  By default, Debian
 # installs a self-signed certificate.  This is useful for testing, but you
@@ -15,9 +15,9 @@
 # domains in dovecot-openssl.cnf
 #
 # Preferred permissions: root:root 0444
-ssl_server_cert_file = /etc/dovecot/private/dovecot.pem
+ssl_server_cert_file = /etc/letsencrypt/live/mail.example.com/fullchain.pem
 # Preferred permissions: root:root 0400
-ssl_server_key_file = /etc/dovecot/private/dovecot.key
+ssl_server_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem

 # If key file is password protected, give the password here. Alternatively
 # give it when starting dovecot with -p parameter. Since this file is often
@@ -44,7 +44,7 @@

 # SSL protocols to use.  Debian systems specify TLSv1.2 by default, which should
 # be reasonbly secure and compatible with existing clients.
-ssl_min_protocol = TLSv1.2
+ssl_min_protocol = TLSv1.3
 # Diffie-Hellman parameters are no longer required and should be phased out.
 # They do not work with ECDH(E) and require DH(E) ciphers.
 #ssl_server_dh_file = /etc/dovecot/dh.pem
```

### Mail delivery

```diff
# diff --color --unified 15-lda.conf.ucf-dist 15-lda.conf
--- 15-lda.conf.ucf-dist
+++ 15-lda.conf
@@ -4,11 +4,11 @@

 # Address to use when sending rejection mails.
 # Default is postmaster@%d. %d expands to recipient domain.
-#postmaster_address =
+postmaster_address = postmaster@example.com

 # Hostname to use in various parts of sent mails (e.g. in Message-Id) and
 # in LMTP replies. Default is the system's real hostname@domain.
-#hostname =
+hostname = smtp.example.com

 # If user is over quota, return with temporary failure instead of
 # bouncing the mail.
@@ -29,7 +29,7 @@
 #rejection_reason = Your message to <%t> was automatically rejected:%n%r

 # Delimiter character between local-part and detail in email address.
-#recipient_delimiter = +
+recipient_delimiter = +

 # Header where the original recipient address (SMTP's RCPT TO: address) is taken
 # from if not available elsewhere. With dovecot-lda -a parameter overrides this.
@@ -37,14 +37,14 @@
 #lda_original_recipient_header =

 # Should saving a mail to a nonexistent mailbox automatically create it?
-#lda_mailbox_autocreate = no
+lda_mailbox_autocreate = yes

 # Should automatically created mailboxes be also automatically subscribed?
 #lda_mailbox_autosubscribe = no

 protocol lda {
   # Boolean list of plugins to load
-  #mail_plugins {
-  #   sieve = yes
-  #}
+  mail_plugins {
+     sieve = yes
+  }
 }
```

### IMAP

```diff
# diff --color --unified 20-imap.conf.ucf-dist 20-imap.conf                      :(
--- 20-imap.conf.ucf-dist
+++ 20-imap.conf
@@ -98,9 +98,9 @@

 protocol imap {
   # Space separated list of plugins to load (default is global mail_plugins).
-  #mail_plugins {
-  #  imap_sieve = yes
-  #}
+  mail_plugins {
+    imap_sieve = yes
+  }

   # Maximum number of IMAP connections allowed for a user from each IP address.
   # NOTE: The username is compared case-sensitively.
```

### Quota

```diff
# diff --color --unified 90-quota.conf.ucf-dist 90-quota.conf                      :(
--- 90-quota.conf.ucf-dist
+++ 90-quota.conf
@@ -14,13 +14,13 @@
 # from userdb. It's also possible to give mailbox-specific limits, for example
 # to give additional 100 MB when saving to Trash:

-#mail_plugins {
-#  quota = yes
-#}
+mail_plugins {
+  quota = yes
+}

-#quota "User quota" {
-#   storage_size = 1G
-#}
+quota user {
+   storage_size = 1G
+}
 #
 #namespace inbox {
 #   mailbox Trash {
```

### Sieve

```diff
# diff --color --unified 90-sieve.conf.ucf-dist 90-sieve.conf
--- 90-sieve.conf.ucf-dist
+++ 90-sieve.conf
@@ -8,11 +8,11 @@
 # See https://doc.dovecot.org/latest/core/plugins/sieve.html
 
 # Personal sieve script location
-#sieve_script personal {
-#  driver = file
-#  path = ~/sieve
-#  active_path = ~/.dovecot.sieve
-#}
+sieve_script personal {
+  driver = file
+  path = ~/sieve
+  active_path = ~/.dovecot.sieve
+}
 
 # Default sieve script location
 #sieve_script default {
```

To enable sieve filter rules, put the [Sieve](/sieve) Link script named .dovecot.sieve into the mail account's virtual home directory.

## Migrating configuration to dovecot 2.4

The update to dovecot 2.4 introduces breaking changes, see https://doc.dovecot.org/main/installation/upgrade/2.3-to-2.4.html.

In this section, we describe the important changes for our setup and suggest the following update path:

1. Update your configuration files as described below, while the old setup with dovecot prior 2.4 is still running.
2. Update dovecot, probably with a debian release update, and use your configuration files from the first step to apply their changes to the deployed default config of your system.

Following this plan you'll get a setup that has minimal difference to the default setup and is therefore ready for the next update.
Alternatively you might want to use [the official automatic updater](https://dovecot.org/upgrader/).

First you can delete the following configuration files:
```
10-director.conf
10-tcpwrapper.conf
90-plugin.conf
auth-vpopmail.conf.ext
```

Starting with dovecot 2.4 the **first** paramenters must be `dovecot_config_version` and `dovecot_storage_version`.

```diff
# dovecot.conf
@@ -20,6 +20,9 @@
 # options. The paths listed here are for configure --prefix=/usr
 # --sysconfdir=/etc --localstatedir=/var

+dovecot_config_version = 2.4.0
+dovecot_storage_version = 2.4.0
+
 # Enable installed protocols
 !include_try /usr/share/dovecot/protocols.d/*.protocol
```

### Authentication / Virtual Users

```diff
# 10-auth.conf
@@ -110,18 +110,24 @@
 # duplicating the system users into virtual database.
 #
 # <doc/wiki/PasswordDatabase.txt>
-passdb {
+passdb passwd-file {
   driver = passwd-file
-  args = /etc/dovecot/passwd
+  passwd_file_path = /etc/dovecot/passwd
 }
 #
 # User database specifies where mails are located and what user/group IDs
 # own them. For single-UID configuration use "static" userdb.
 #
 # <doc/wiki/UserDatabase.txt>
-userdb {
+userdb static {
   driver = static
-  args = uid=vmail gid=mail home=/var/vmail/%d/%n allow_all_users=yes
+
+  fields {
+    allow_all_users = yes
+    gid = mail
+    home = /var/vmail/%{user|domain}/%{user|username}
+    uid = vmail
+  }
 }
 
 #!include auth-deny.conf.ext
```

Note that the placeholders have changed, e.g. `%d` to `%{user | domain}` and `%n` to `%{user | username}`. (The curly brackets are important.)

```diff
# auth-system.conf.ext
@@ -7,7 +7,7 @@
 # PAM is typically used with either userdb passwd or userdb static.
 # REMEMBER: You'll need /etc/pam.d/dovecot file created for PAM
 # authentication to actually work. <doc/wiki/PasswordDatabase.PAM.txt>
-passdb {
+passdb pam {
   driver = pam
   # [session=yes] [setcred=yes] [failure_show_msg=yes] [max_requests=<n>]
   # [cache_key=<key>] [<service name>]
@@ -46,7 +46,7 @@

 # System users (NSS, /etc/passwd, or similar). In many systems nowadays this
 # uses Name Service Switch, which is configured in /etc/nsswitch.conf.
-userdb {
+userdb passwd {
   # <doc/wiki/AuthDatabase.Passwd.txt>
   driver = passwd
   # [blocking=no]
```

### Mail User

```diff
# 10-mail.conf
@@ -27,7 +27,9 @@
 #
 # <doc/wiki/MailLocation.txt>
 #
-mail_location = maildir:~/mail
+mail_driver = maildir
+mail_path = %{home}/mail

 # If you need to set multiple mailbox locations or want to change default
 # namespace settings, you can do it by defining namespace sections.
@@ -214,7 +216,7 @@

 # Space separated list of plugins to load for all services. Plugins specific to
 # IMAP, LDA, etc. are added to this list in their own .conf files.
-mail_plugins = $mail_plugins quota
+#mail_plugins = $mail_plugins quota

 ##
 ## Mailbox handling optimizations
```

Especially the `mail_location` was split into the parameters `mail_driver` and `mail_path`.

### Postfix Authentication via Dovecot

```diff
# 10-master.conf
@@ -105,7 +105,7 @@

   # Postfix smtp-auth
   unix_listener /var/spool/postfix/private/auth {
-    mode = 0666
+    mode = 0660
     user = postfix
     group = postfix
   }
```

### SSL

```diff
# 10-ssl.conf
@@ -3,14 +3,14 @@
 ##
 
 # SSL/TLS support: yes, no, required. <doc/wiki/SSL.txt>
-ssl = yes
+ssl = required
 
 # PEM encoded X.509 SSL/TLS certificate and private key. They're opened before
 # dropping root privileges, so keep the key file unreadable by anyone but
 # root. Included doc/mkcert.sh can be used to easily generate self-signed
 # certificate, just make sure to update the domains in dovecot-openssl.cnf
-ssl_cert = </etc/letsencrypt/live/jensbrandt.de-0002/fullchain.pem
-ssl_key = </etc/letsencrypt/live/jensbrandt.de-0002/privkey.pem
+ssl_server_cert_file = /etc/letsencrypt/live/jensbrandt.de-0002/fullchain.pem
+ssl_server_key_file = /etc/letsencrypt/live/jensbrandt.de-0002/privkey.pem
 
 # If key file is password protected, give the password here. Alternatively
 # give it when starting dovecot with -p parameter. Since this file is often
@@ -52,14 +52,14 @@
 # Generate new params with `openssl dhparam -out /etc/dovecot/dh.pem 4096`
 # Or migrate from old ssl-parameters.dat file with the command dovecot
 # gives on startup when ssl_dh is unset.
-ssl_dh = </usr/share/dovecot/dh.pem
+ssl_server_dh_file = /usr/share/dovecot/dh.pem
 
 # Minimum SSL protocol version to use. Potentially recognized values are SSLv3,
 # TLSv1, TLSv1.1, TLSv1.2 and TLSv1.3, depending on the OpenSSL version used.
 #
 # Dovecot also recognizes values ANY and LATEST. ANY matches with any protocol
 # version, and LATEST matches with the latest version supported by library.
-ssl_min_protocol = TLSv1.2
+ssl_min_protocol = TLSv1.3
 
 # SSL ciphers to use, the default is:
 #ssl_cipher_list = ALL:!kRSA:!SRP:!kDHd:!DSS:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!PSK:!RC4:!ADH:!LOW@STRENGTH
```

Note that the `<` symbol isn't needed for the ssl parameters anymore.

### Mail delivery

```diff
# 15-lda.conf
@@ -44,6 +44,8 @@

 protocol lda {
   # Space separated list of plugins to load (default is global mail_plugins).
-  mail_plugins = $mail_plugins sieve
+  mail_plugins {
+    sieve = yes
+  }
   mail_fsync = optimized
 }
```

### IMAP

```diff
# 20-imap.conf
@@ -91,7 +91,9 @@

 protocol imap {
   # Space separated list of plugins to load (default is global mail_plugins).
-  mail_plugins = $mail_plugins imap_quota
+  mail_plugins {
+    imap_quota = yes
+  }

   # Maximum number of IMAP connections allowed for a user from each IP address.
   # NOTE: The username is compared case-sensitively.
```

### Quota

```diff
# 90-quota.conf
@@ -15,8 +15,8 @@
 # to give additional 100 MB when saving to Trash:
 
 plugin {
-  qouta = dict:User qouta::file:%h/mail/dovecot-qouta
-  qouta_rule = *:storage=1000MB
+  #qouta = dict:User qouta::file:%h/mail/dovecot-qouta
+  #qouta_rule = *:storage=1000MB
   #quota_rule = *:storage=1G
   #quota_rule2 = Trash:storage=+100M
 
@@ -29,6 +29,11 @@
   #quota_max_mail_size = 100M
 }
 
+quota user {
+  driver = count
+}
+quota_rule = *:storage=1000MB
+
 ##
 ## Quota warnings
 ##
```

### Sieve

```diff
# 90-sieve.conf
@@ -36,7 +36,7 @@
   # active script symlink is located.
   # For other types: use the ';name=' parameter to specify the name of the
   # default/active script.
-  sieve = file:~/sieve;active=~/.dovecot.sieve
+  #sieve = file:~/sieve;active=~/.dovecot.sieve

   # The default Sieve script when the user has none. This is the location of a
   # global sieve script file, which gets executed ONLY if user's personal Sieve
@@ -203,3 +203,9 @@
   # the source line numbers.
   #sieve_trace_addresses = no
 }
+
+sieve_script personal {
+  active_path = ~/.dovecot.sieve
+  driver = file
+  path = ~/sieve
+}
```
