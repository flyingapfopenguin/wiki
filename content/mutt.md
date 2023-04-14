---
title: "mutt"
date: 2023-04-06T22:54:01+02:00
---

Projectpage: http://www.mutt.org/.

## Installation

On Gentoo we use the following USE-Flags for mutt.

```
mail-client/mutt -hcache gpgme imap smtp sasl
```

```bash
apt install mutt cyrus-sasl2 # on Debian
emerge -av mail-client/mutt # on Gentoo
pacman -S mutt cyrus-sasl-gssapi # on Arch
```

## Configuration

The configuration is placed in ~/.mutt.

We start with the main configuration file ~/.mutt/muttrc

```
# character set on messages that we send
set send_charset="utf-8"
# if there is no character set given on incoming messages, it is probably windows
set assumed_charset="iso-8859-1"
  
# make sure Vim knows Mutt is a mail client and that we compose an UTF-8 encoded message
set editor="vim -c 'set syntax=mail ft=mail enc=utf-8'"
  
# just scroll one line instead of full page
set menu_scroll=yes
  
# we want to see some MIME types inline, see below this code listing for explanation
#auto_view application/msword
#auto_view application/pdf
  
# make default search pattern to search in To, Cc and Subject
set simple_search="~f %s | ~C %s | ~s %s"
  
# threading preferences, sort by threads
#set sort=threads
#set strict_threads=yes
set sort=reverse-date-received

#auto_view text/html
alternative_order text/plain text/html
set mailcap_path = ~/.mutt/mailcap
set smart_wrap = yes
set markers = yes
set pager_stop = yes

set date_format="%d.%m.%Y %T"
set index_format="%4C | %Z [%d] %-30.30F (%-4.4c) %s"

# show spam score (from SpamAssassin only) when reading a message
spam "X-Spam-Score: ([0-9\\.]+).*" "SA: %1"
  
# do not show all headers, just a few
ignore          *
unignore        From To Cc Bcc Date X-Spam-Status Subject
# and in this order
unhdr_order     *
hdr_order       Date: From: To: Cc: Bcc: Subject: X-Spam-Status:
  
# brighten up stuff with colours, for more colouring examples see:
# http://aperiodic.net/phil/configs/mutt/colors
color normal      white          black
color hdrdefault  green          default
color quoted      green          default
color quoted1     yellow         default
color quoted2     red            default
color signature   cyan           default
color indicator   brightyellow   red
color error       brightred      default
color status      brightwhite    blue
color tree        brightmagenta  black
color tilde       blue           default
color attachment  brightyellow   default
color markers     brightred      default
color message     white          black
color search      brightwhite    magenta
color bold        brightyellow   default
# if you don't like the black progress bar at the bottom of the screen,
# comment out the following line
color progress    white          black
  
# personality settings
set realname = <Real Name> # TODO
set from = <email-address> # TODO
set use_from=yes
set ssl_force_tls=yes
set ssl_starttls=yes
#alternates "andrew@mail.server|andrew.dalziel@mail.server"
# this file must exist, and contains your signature, comment it out if
# you don't want a signature to be used
#set signature = ~/.signature
  
# aliases (sort of address book)
#source ~/.aliases
#set alias_file=~/.mutt/aliases     # aliases-file is there
#set sort_alias= address
#set sort_alias= alias
set sort_alias= unsorted
set reverse_alias=yes
#source $alias_file
set query_command="abook --mutt-query '%s'"  # address book
  
# IMAP connection settings
set mail_check=60
set imap_keepalive=300
  
# IMAP account settings
set imap_user=<imap-user> # TODO e.g. mail address
set imap_pass=<password> # TODO
set folder=<imap root> # TODO e.g. imaps://imap.domain.com/
#set spoolfile=imaps://andy@imap.mail.server/
#set record=imaps://andy@imap.mail.server/Sent
#set postponed=imaps://andy@imap.mail.server/Drafts
set spoolfile=+INBOX
set postponed=+Drafts
set imap_check_subscribed
set message_cachedir=~/.mutt/cache
unset imap_passive
  
# use headercache for IMAP (make sure this is a directory for performance!)
#set header_cache=/var/tmp/.mutt
  
# mailboxes we want to monitor for new mail
#mailboxes "="
#mailboxes "=Lists"
mailboxes =INBOX
  
# mailing lists we are on (these are regexps!)
#subscribe "gentoo-.*@gentoo\\.org"
  
# SMTP mailing configuration (for sending mail)
set imap_authenticators="cram-md5" # TODO or plain?
set smtp_url=<smtp server> # TODO e.g. smtp://$imap_user@smtp.domain.com:587"
#set smtp_authenticators = "login"
set smtp_pass="$imap_pass"
set record=+Sent

#Sidebar
source ~/.mutt/sidebar.muttrc

# Further (optional tweaking)
set edit_headers=yes

# keybinds
source ~/.mutt/keybinds.muttrc

# gpg setup
#set crypt_use_gpgme=yes
#source ~/.mutt/gpg.muttrc
```

The sidebar is configured in ~/.mutt/sidebar.muttrc

```
# shamelessly copied from
# http://www.lunar-linux.org/index.php?option=com_content&task=view&id=44

# set up the sidebar, default not visible
set sidebar_width=30
set sidebar_visible=yes
#set sidebar_format = "%B%* %n %S"
#set sidebar_delim='|'

# which mailboxes to list in the sidebar
mailboxes =INBOX

# color of folders with new mail
color sidebar_new brightgreen default

# sort folder by name
set sidebar_sort_method=name

# ctrl-n, ctrl-p to select next, prev folder
# ctrl-o to open selected folder
bind index - sidebar-prev
bind index + sidebar-next
bind pager - sidebar-prev
bind pager + sidebar-next
bind index \CP sidebar-prev-new
bind index \CN sidebar-next-new
bind pager \CP sidebar-prev-new
bind pager \CN sidebar-next-new
bind index \CO sidebar-open
bind pager \CO sidebar-open

# I don't need these. just for documentation purposes. See below.
# sidebar-scroll-up
# sidebar-scroll-down

# b toggles sidebar visibility
macro index b '<enter-command>toggle sidebar_visible<enter>'
macro pager b '<enter-command>toggle sidebar_visible<enter>'

# Remap bounce-message function to "B"
bind index B bounce-message
```

Your keybinds are definded in ~/.mutt/keybinds.muttrc

```
# ~/.mutt/keybinds.muttrc
bind pager <Up>  "previous-line"   # scroll up one line
bind pager <Down>  "next-line"   # scroll up one line

# Go to folder...
# source https://ryanlue.com/posts/2017-05-21-mutt-the-vim-way
macro index,pager gi "<change-folder>=INBOX<enter>"       "open inbox"
macro index,pager gd "<change-folder>=Drafts<enter>"      "open drafts"
macro index,pager gs "<change-folder>=Sent<enter>"        "open sent"
macro index,pager gt "<change-folder>$trash<enter>"       "open trash"
macro index,pager gf "<change-folder>?"                   "open mailbox..."
```

If you want to use pgp, configure it in ~/.mutt/gpg.muttrc and source this file in the main config.

```
# vim:syn=muttrc                                                                
##

set pgp_default_key=<keyID> # TODO

# Decode application/pgp attachments like so:
set pgp_decode_command="/usr/bin/gpg %?p?--passphrase-fd 0? --no-verbose --batch --output - %f"

# And use this to verify pgp signatures:
set pgp_verify_command="/usr/bin/gpg --no-verbose --batch --output - --verify %s %f"

# How to decrypt pgp encrypted messages:
set pgp_decrypt_command="/usr/bin/gpg --passphrase-fd 0 --no-verbose --batch --output - %f"

# How to pgp sign a message:
set pgp_sign_command="/usr/bin/gpg --no-verbose --batch --output - --passphrase-fd 0 --armor --detach-sign --textmode %?a?-u %a? %f"

# How to pgp clearsign a message:
set pgp_clearsign_command="/usr/bin/gpg --no-verbose --batch --output - --passphrase-fd 0 --armor --textmode --clearsign %?a?-u %a? %f"

# Import a pgp key from a message into my public keyring as follows:
set pgp_import_command="/usr/bin/gpg --no-verbose --import -v %f"

# Use this to export a key from my public keyring:
set pgp_export_command="/usr/bin/gpg --no-verbose --export --armor %r"

# Verify key information (from the key selection menu):
set pgp_verify_key_command="/usr/bin/gpg --verbose --batch --fingerprint --check-sigs %r"

# List my public keyring like so:
set pgp_list_pubring_command="/usr/bin/gpg --no-verbose --batch --with-colons --list-keys %r" 

# List my private keyring like so:
set pgp_list_secring_command="/usr/bin/gpg --no-verbose --batch --with-colons --list-secret-keys %r" 

# Automatically test signature from incoming messages
set crypt_verify_sig=yes

# Automatically sign outgoing messages
set pgp_autosign=yes

# automatically encrypt emails to recipients for whom I have the key
set crypt_opportunistic_encrypt=yes

# Timeout (in seconds) for cached passphrases:
set pgp_timeout=10

# Text to show before a good signature:
set pgp_good_sign="gpg: (Korrekte Signatur von|Good signature from) "

# Commands to encrypt (and sign) messages:
set pgp_encrypt_only_command="pgpewrap gpg --batch --quiet --no-verbose --output - --encrypt --textmode --armor --always-trust -- -r %r -- %f"
set pgp_encrypt_sign_command="pgpewrap gpg --passphrase-fd 0 --batch --quiet --no-verbose --textmode --output - --encrypt --sign %?a?-u %a? --armor --always-trust -- -r %r -- %f"

# Encrypt messages with my own key:
set pgp_self_encrypt=yes
```

To configure the address book save your adresses (one per line) in ~/.mutt/aliases and set and source this file in the main config.

```
alias <nick> <email-address>
```