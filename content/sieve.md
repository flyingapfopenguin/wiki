---
title: "Sieve"
date: 2023-04-06T22:54:01+02:00
---

This article was initially tested on Debian 9 (stretch) and updated to Debian 10 (buster).

This article is shamelessly copied from my friend edr, see https://wiki.rochefort.de/sysadmin/mailserver/sieve.

Sieve scripts belong into the virtual users homedir and should be named .dovecot.sieve

To do some basic sorting for tags and / or spam flags use scripts similiar to this:

```
require "fileinto";

#if header :contains "subject" ["foo", "bar"] {
#      fileinto "foofolder";
#}

#if header :matches "Subject" ["*foo*","*bar*"]) {
#      fileinto "foofolder";
#}

if header :contains "X-Spam-Flag" "Yes" {
    fileinto "Spam";
    stop;
}
```

Note that this assumes that you have a working [Spamassassin](/spamassassin) setup.
