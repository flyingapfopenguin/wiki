---
title: "Spamassassin"
date: 2023-04-06T22:54:01+02:00
---

This article is shamelessly copied (and a little modified) from my friend edr, see https://wiki.rochefort.de/sysadmin/mailserver/spamassassin.

Largely based on https://www.christianroessler.net/tech/2015/spamassassin-dovecot-postfix.html.

## Installation

```bash
apt install spamassassin spamc dovecot-antispam
```

## Configuration

Enable virtual user support for the trained Bayes databases so that each account can have its own spam filter training data.

```diff
--- a/data/etc/default/spamassassin
+++ b/data/etc/default/spamassassin
@@ -17,7 +17,7 @@ ENABLED=0
 # make sure --max-children is not set to anything higher than 5,
 # unless you know what you're doing.
 
-OPTIONS="--create-prefs --max-children 5 --helper-home-dir"
+OPTIONS="--create-prefs --max-children 5 --helper-home-dir --virtual-config-dir=/var/vmail/%d/%l/spamassassin --allow-tell --timeout-child 30 --username vmail -x"
 
 # Pid file
 # Where should spamd write its PID to file? If you use the -u or
@@ -31,4 +31,4 @@ PIDFILE="/var/run/spamd.pid"
 # Cronjob
 # Set to anything but 0 to enable the cron job to automatically update
 # spamassassin's rules on a nightly basis
-CRON=0
+CRON=1
```

```diff
--- a/etc/spamassassin/local.cf
+++ b/etc/spamassassin/local.cf
@@ -10,6 +10,7 @@
 #   Add *****SPAM***** to the Subject header of spam e-mails
 #
 # rewrite_header Subject *****SPAM*****
+add_header all Status _YESNO_, score=_SCORE_ required=_REQD_ tests=_TESTS_ autolearn=_AUTOLEARN_ version=_VERSION_
 
 
 #   Save spam messages as a message/rfc822 MIME attachment instead of
@@ -31,25 +32,25 @@
 
 #   Set the threshold at which a message is considered spam (default: 5.0)
 #
-# required_score 5.0
+required_score 3.0
 
 
 #   Use Bayesian classifier (default: 1)
 #
-# use_bayes 1
+use_bayes 1
 
 
 #   Bayesian classifier auto-learning (default: 1)
 #
-# bayes_auto_learn 1
+bayes_auto_learn 1
 
 
 #   Set headers which may provide inappropriate cues to the Bayesian
 #   classifier
 #
-# bayes_ignore_header X-Bogosity
-# bayes_ignore_header X-Spam-Flag
-# bayes_ignore_header X-Spam-Status
+bayes_ignore_header X-Bogosity
+bayes_ignore_header X-Spam-Status
+bayes_ignore_header X-Spam-Flag
 
 #   Some shortcircuiting, if the plugin is enabled
```

## Training the filter

In order to conveniently train the Bayes database, you can use dedicated folders for spam and ham and use a cronjob to periodically feed their contents to sa-learn.

A script that does that just could look like this:

```bash
#!/bin/bash
source /usr/local/include/lock_process.sh
lock "SA-LEARN"
MAIL_ROOT=/var/vmail
VMAIL_USER=vmail
SA_SPAM_FOLDER=.sa_spam
SA_HAM_FOLDER=.sa_ham
SPAM_FOLDER=.Spam
HAM_FOLDER=.

function learn() {
    local sa_type=$1
    local db_file=$2
    local src_dir=$3
    local dst_dir=$4
    if [ -d $src_dir ]; then
        [ -z "$(ls -A $src_dir)" ] && return
        [ -d $dst_dir ] || return
        for mail in $src_dir/*; do
            sa-learn --$sa_type -u debian-spamd --dbpath $db_file --dir $mail
            mv -v $mail $dst_dir
        done
    fi
}

for DOMAIN_DIR in $MAIL_ROOT/*; do
    for USER_DIR in $DOMAIN_DIR/*; do
        DB_PATH=$USER_DIR/spamassassin 
        DB_FILE=$DB_PATH/bayes 
        learn spam ${DB_FILE} $USER_DIR/mail/$SA_SPAM_FOLDER/cur $USER_DIR/mail/$SPAM_FOLDER/cur
        learn ham  ${DB_FILE} $USER_DIR/mail/$SA_HAM_FOLDER/cur  $USER_DIR/mail/$HAM_FOLDER/cur
        [ -d ${DB_PATH} ] && chown -R $VMAIL_USER ${DB_PATH}
    done
done
unlock "SA-LEARN"
```

Things to note:

* In order to prevent concurrent runs this script uses the lock function shown in the article [Locking mechanism in shell scripts](https://wiki.rochefort.de/bash/lock-helper)
* The script assumes that the folders Spam, sa\_spam and sa\_ham already exist
* The script assumes the virtual user directory structure presented in [Creating mail accounts](/postfix/#creating-mail-accounts)

Now you can use your mail client to move mail to sa\_spam or sa\_ham to teach spamassassin to recognize them as spam and ham respectively. 
