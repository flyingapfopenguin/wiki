---
title: "Apache"
date: 2023-04-06T22:54:01+02:00
---

see: https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-debian-9.

This article was tested on Debian 9 (stretch).

## Installation

```bash
apt install apache2
```

## Configuration

### Non-SSL for static content

```
<VirtualHost _default_:80>
  ServerName brandt-george.de
  ServerAlias www.brandt-george.de
  DocumentRoot /var/www/html/brandt-george_de
  Protocols h2 http/1.1
  <Directory /var/www/html/brandt-george_de>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
  </Directory>
</VirtualHost>
```

### SSL for static content

```bash
a2enmod headers
a2enmod ssl
```

Configure TLS in /etc/apache2/mods-available/ssl.conf:

```
SSLCipherSuite HIGH:!aNULL
SSLHonorCipherOrder on
SSLProtocol -all +TLSv1.2 +TLSv1.3
```

```
<VirtualHost _default_:80>
  ServerName brandt-george.de
  ServerAlias www.brandt-george.de
  Redirect permanent / https://www.brandt-george.de/
</VirtualHost>

<VirtualHost _default_:443>
  ServerName brandt-george.de:443
  ServerAlias www.brandt-george.de
  DocumentRoot /var/www/html/brandt-george_de
  Protocols h2 http/1.1
  SSLEngine on
  SSLCertificateFile    /etc/letsencrypt/live/brandt-george.de/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/brandt-george.de/privkey.pem
  # active HSTS -- requiers mod_headers!
  Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains"
  Header always set X-Frame-Options: DENY
  Header always set X-Content-Type-Options: nosniff
  Header always set Content-Security-Policy: "default-src 'self'"
  Header always set Referrer-Policy: same-origin
  <Directory /var/www/html/brandt-george_de>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
  </Directory>
</VirtualHost>
```

### SnappyMail Webmailer

see [SnappyMail Webmailer](/snappymail).

For SnappyMail we need a different Content-Security-Policy:

```
Header always set Content-Security-Policy "default-src https:; img-src https: data:; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'"
```

### Nextcloud

see [Nextcloud](/nextcloud).

```
<VirtualHost _default_:443>
  ServerName cloud.brandt-george.de:443
  ServerAlias cloud.brandt-george.de
  DocumentRoot /var/www/html/cloud_brandt-george_de
  SSLEngine on
  SSLCertificateFile    /etc/letsencrypt/live/brandt-george.de/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/brandt-george.de/privkey.pem
  <Directory /var/www/html/cloud_brandt-george_de>
    Options +FollowSymlinks
    AllowOverride All
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
    <IfModule mod_headers.c>
      Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
    </IfModule>
    SetEnv HOME /var/www/html/cloud_brandt-george_de/
    SetEnv HTTP_HOME /var/www/html/cloud_brandt-george_de/
  </Directory>
</VirtualHost>
```

### Wordpress

see [Wordpress](/wordpress).

```
<VirtualHost _default_:443>
        ServerName brandt-george.de:443
        ServerAlias www.brandt-george.de
        DocumentRoot /var/www/html/brandt-george_de
        Protocols h2 http/1.1
        SSLEngine on
        SSLCertificateFile    /etc/letsencrypt/live/brandt-george.de/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/brandt-george.de/privkey.pem
        # HSTS einrichten -- erfordert mod_headers!
        #Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains"
        #Header always set X-Frame-Options: DENY
        #Header always set X-Content-Type-Options: nosniff
        ##Header always set Content-Security-Policy: "default-src 'self'"
        #Header always set Referrer-Policy: same-origin
        <Directory /var/www/html/brandt-george_de>
            Options FollowSymLinks
            AllowOverride Limit Options FileInfo
            DirectoryIndex index.php
            Require all granted
        </Directory>
</VirtualHost>
```
