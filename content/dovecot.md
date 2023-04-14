---
title: "Dovecot"
date: 2023-04-06T22:54:01+02:00
---

This article was initially tested on Debian 9 (stretch) and updated to Debian 10 (buster).

This article is shamelessly copied (and a little modified) from my friend edr, see https://wiki.rochefort.de/sysadmin/mailserver/dovecot.

## Overview

Dovecot gives you some additional in comparison to [Postfix](/postfix) to deliver incoming mail. It also allows you use mail clients to access your mail via IMAP.

## Installation

```bash
apt install dovecot-imapd dovecot-sieve
```

## Configuration - General

The dovecot configuration is split up into many individual files that live in /etc/dovecot/conf.d To save space and keep the content of this page within reasonable size, I am only showing a diff of these files to their respective original versions.

## Configuration - Authentication / Virtual Users

```diff
diff --git a/etc/dovecot/conf.d/10-auth.conf b/etc/dovecot/conf.d/10-auth.conf
--- a/etc/dovecot/conf.d/10-auth.conf
+++ b/etc/dovecot/conf.d/10-auth.conf
@@ -97,7 +97,9 @@
 #   plain login digest-md5 cram-md5 ntlm rpa apop anonymous gssapi otp skey
 #   gss-spnego
 # NOTE: See also disable_plaintext_auth setting.
-auth_mechanisms = plain login cram-md5
+auth_mechanisms = plain cram-md5
+auth_verbose = yes
 
 ##
 ## Password and user databases
@@ -110,11 +112,19 @@ auth_mechanisms = plain
 # duplicating the system users into virtual database.
 #
 # <doc/wiki/PasswordDatabase.txt>
+passdb {
+  driver = passwd-file
+  args = /etc/dovecot/passwd
+}
 #
 # User database specifies where mails are located and what user/group IDs
 # own them. For single-UID configuration use "static" userdb.
 #
 # <doc/wiki/UserDatabase.txt>
+userdb {
+  driver = static
+  args = uid=vmail gid=mail home=/var/vmail/%d/%n allow_all_users=yes
+}
 
 #!include auth-deny.conf.ext
 #!include auth-master.conf.ext
```

To define the user / password database, the above config refers to /etc/dovecot/passwd This file must be defined using the following syntax:

```
user1@example.com:{CRAM-MD5}password1::::::
user2@example.com:{CRAM-MD5}password2::::::
```

The password hashes can be generated via:
```bash
doveadm pw -s CRAM-MD5
```

The path to the home directory contains placeholders. %d stands for the domain name, %n for the username.

## Configuration - Mail Plugins

```diff
diff --git a/etc/dovecot/conf.d/10-director.conf b/etc/dovecot/conf.d/10-director.conf
--- a/etc/dovecot/conf.d/10-director.conf
+++ b/etc/dovecot/conf.d/10-director.conf
@@ -58,4 +58,5 @@ service pop3-login {
 # Enable director for LMTP proxying:
 protocol lmtp {
   #auth_socket_path = director-userdb
+  mail_plugins = $mail_plugins sieve
 }
```

The sieve plugin adds support for scripting features to e.g. sort incoming mail.

## Configuration - Logging

```diff
diff --git a/etc/dovecot/conf.d/10-logging.conf b/etc/dovecot/conf.d/10-logging.conf
--- a/etc/dovecot/conf.d/10-logging.conf
+++ b/etc/dovecot/conf.d/10-logging.conf
@@ -4,10 +4,10 @@
 
 # Log file to use for error messages. "syslog" logs to syslog,
 # /dev/stderr logs to stderr.
-#log_path = syslog
+log_path = /var/log/dovecot.log
 
 # Log file to use for informational messages. Defaults to log_path.
-#info_log_path = 
+info_log_path = /var/log/dovecot-info.log
 # Log file to use for debug messages. Defaults to info_log_path.
 #debug_log_path =
```

For dovecot to be able to write to the logfiles that are referred to in the above config, you need to

```
touch /var/log/dovecot.log
touch /var/log/dovecot-info.log
chown vmail:mail /var/log/dovecot*.log
```

## Configuration - Mail User and Location

```diff
diff --git a/etc/dovecot/conf.d/10-mail.conf b/etc/dovecot/conf.d/10-mail.conf
--- a/etc/dovecot/conf.d/10-mail.conf
+++ b/etc/dovecot/conf.d/10-mail.conf
@@ -27,7 +27,8 @@
 #
 # <doc/wiki/MailLocation.txt>
 #
-mail_location = mbox:~/mail:INBOX=/var/mail/%u
+# mail_location = mbox:~/mail:INBOX=/var/mail/%u
+mail_location = maildir:~/mail
 
 # If you need to set multiple mailbox locations or want to change default
 # namespace settings, you can do it by defining namespace sections.
@@ -158,15 +159,15 @@ namespace inbox {
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
@@ -198,7 +199,7 @@ namespace inbox {
 
 # Space separated list of plugins to load for all services. Plugins specific to
 # IMAP, LDA, etc. are added to this list in their own .conf files.
-#mail_plugins = 
+mail_plugins = $mail_plugins quota
 
 ##
 ## Mailbox handling optimizations
```

The mail location is specified relative to the virtual user's home directory as specified in [Configuration - Authentication / Virtual Users](/dovecot#configuration---authentication--virtual-users).

The user id and group id must be in line with the system account you created in [Creating mail accounts](/postfix#creating-mail-accounts).

## Postfix Authentication via Dovecot

```diff
diff --git a/data/etc/dovecot/conf.d/10-master.conf b/data/etc/dovecot/conf.d/10-master.conf
--- a/data/etc/dovecot/conf.d/10-master.conf
+++ b/data/etc/dovecot/conf.d/10-master.conf
@@ -86,10 +86,11 @@ service auth {
   # To give the caller full permissions to lookup all users, set the mode to
   # something else than 0666 and Dovecot lets the kernel enforce the
   # permissions (e.g. 0777 allows everyone full permissions).
-  unix_listener auth-userdb {
-    #mode = 0666
-    #user = 
-    #group = 
+  #unix_listener auth-userdb {
+  unix_listener /var/spool/postfix/private/auth {
+    mode = 0666
+    user = postfix
+    group = postfix
   }
 
   # Postfix smtp-auth
```

## Configuration - SSL

```diff
diff --git a/etc/dovecot/conf.d/10-ssl.conf b/etc/dovecot/conf.d/10-ssl.conf
--- a/etc/dovecot/conf.d/10-ssl.conf
+++ b/etc/dovecot/conf.d/10-ssl.conf
@@ -3,7 +3,7 @@
 ##
 
 # SSL/TLS support: yes, no, required. <doc/wiki/SSL.txt>
-ssl = yes
+ssl = required
 
 # PEM encoded X.509 SSL/TLS certificate and private key. They're opened before
 # dropping root privileges, so keep the key file unreadable by anyone but
@@ -11,6 +11,8 @@ ssl = no
 # certificate, just make sure to update the domains in dovecot-openssl.cnf
 #ssl_cert = </etc/dovecot/dovecot.pem
 #ssl_key = </etc/dovecot/private/dovecot.pem
+ssl_cert = </etc/letsencrypt/live/mail.example.com/fullchain.pem
+ssl_key  = </etc/letsencrypt/live/mail.example.com/privkey.pem
 
 # If key file is password protected, give the password here. Alternatively
 # give it when starting dovecot with -p parameter. Since this file is often
```

## Configuration - Mail delivery

```diff
diff --git a/etc/dovecot/conf.d/15-lda.conf b/etc/dovecot/conf.d/15-lda.conf
--- a/etc/dovecot/conf.d/15-lda.conf
+++ b/etc/dovecot/conf.d/15-lda.conf
@@ -4,11 +4,11 @@
 
 # Address to use when sending rejection mails.
 # Default is postmaster@<your domain>. %d expands to recipient domain.
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
@@ -37,12 +37,12 @@
 #lda_original_recipient_header =
 
 # Should saving a mail to a nonexistent mailbox automatically create it?
-#lda_mailbox_autocreate = no
+lda_mailbox_autocreate = yes
 
 # Should automatically created mailboxes be also automatically subscribed?
 #lda_mailbox_autosubscribe = no
 
 protocol lda {
   # Space separated list of plugins to load (default is global mail_plugins).
-  #mail_plugins = $mail_plugins
+  mail_plugins = $mail_plugins sieve
 }
```

## Configuration - Mailboxes

```diff
diff --git a/etc/dovecot/conf.d/15-mailboxes.conf b/etc/dovecot/conf.d/15-mailboxes.conf
--- a/etc/dovecot/conf.d/15-mailboxes.conf
+++ b/etc/dovecot/conf.d/15-mailboxes.conf
@@ -31,9 +31,9 @@ namespace inbox {
   mailbox Sent {
     special_use = \Sent
   }
-  mailbox "Sent Messages" {
-    special_use = \Sent
-  }
+  #mailbox "Sent Messages" {
+  #  special_use = \Sent
+  #}
 
   # If you have a virtual "All messages" mailbox:
   #mailbox virtual/All {
```

## Configuration - IMAP

```diff
diff --git a/etc/dovecot/conf.d/20-imap.conf b/etc/dovecot/conf.d/20-imap.conf
--- a/etc/dovecot/conf.d/20-imap.conf
+++ b/etc/dovecot/conf.d/20-imap.conf
@@ -53,7 +53,7 @@
 
 protocol imap {
   # Space separated list of plugins to load (default is global mail_plugins).
-  #mail_plugins = $mail_plugins
+  mail_plugins = $mail_plugins imap_quota
 
   # Maximum number of IMAP connections allowed for a user from each IP address.
   # NOTE: The username is compared case-sensitively.
```

## Configuration - Quota

```diff
diff --git a/etc/dovecot/conf.d/90-quota.conf b/etc/dovecot/conf.d/90-quota.conf
--- a/etc/dovecot/conf.d/90-quota.conf
+++ b/etc/dovecot/conf.d/90-quota.conf
@@ -15,6 +15,8 @@
 # to give additional 100 MB when saving to Trash:
 
 plugin {
+  quota = dict:User quota::file:%h/mail/dovecot-quota
+  quota_rule = *:storage=1000MB
   #quota_rule = *:storage=1G
   #quota_rule2 = Trash:storage=+100M
```

## Configuration - Sieve

To enable sieve filter rules, put the [Sieve](/sieve) Link script named .dovecot.sieve into the mail account's virtual home directory.
