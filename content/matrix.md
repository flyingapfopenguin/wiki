---
title: "Matrix"
date: 2023-04-06T22:54:01+02:00
---

This article was tested on Debian 10 (buster) and is based on https://matrix.org/blog/2020/04/06/running-your-own-secure-communication-service-with-matrix-and-jitsi.

In this article we set up a matrix server at matrix.example.com (and optionally an element installation at chat.example.com), while the domain in the IDs will only be example.com.

We assume all three domains have configured DNS and a certificate, see [Let's Encrypt](/letsencrypt).

## Installation

```bash
apt install -y lsb-release wget apt-transport-https
wget -O /usr/share/keyrings/matrix-org-archive-keyring.gpg https://packages.matrix.org/debian/matrix-org-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/matrix-org-archive-keyring.gpg] https://packages.matrix.org/debian/ $(lsb_release -cs) main" >> /etc/apt/sources.list.d/matrix-org.list
apt update
apt install matrix-synapse-py3
```

During the installation set the `Name of the server` to `example.com`.

## Configuration

### PostgreSQL

Set up a postgreSQL database for matrix following https://www.tecmint.com/install-postgresql-database-in-debian-10/, compare config below.

The main configuration file is /etc/matrix-synapse/homeserver.yaml. You might want to configure:

```
web_client_location: https://chat.example.com (if you plan an element installation)

public_baseurl: https://matrix.example.com

use_presence: true

database:
    name: psycopg2
    args:
      user: synapse_user
      password: <password>
      database: synapse
      host: 127.0.0.1
      cp_min: 5
      cp_max: 10

url_preview_enabled: true

url_preview_ip_range_blacklist (complete list)

enable_registration: false
```

### Apache

```
<IfModule mod_ssl.c>
<VirtualHost *:443>
        ServerName matrix.example.com
        DocumentRoot /var/www/html/matrix_example_com/

        #ProxyRequests off
        #ProxyPreserveHost On
        #ProxyVia full

        <Location />
        ProxyPass http://127.0.0.1:8008/ nocanon
        ProxyPassReverse  http://127.0.0.1:8008/
        </Location>

        RequestHeader set X-Forwarded-Proto "https"
        AllowEncodedSlashes NoDecode

        # Configuration of the SSL Certificate
        SSLEngine On
        Include /etc/letsencrypt/options-ssl-apache.conf
        SSLCertificateFile    /etc/letsencrypt/live/example.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
</VirtualHost>
</IfModule>
```

Now https://matrix.example.com should present a "It works! Synapse is running" page.

### Publish subdomain of your matrix server

Clients and other matrix servers should find the following two files at https://example.com:

.well-known/matrix/client:
```
{
    "m.homeserver": {
        "base_url": "https://matrix.example.com:443"
    }
}
```

.well-known/matrix/server:
```
{
    "m.server": "matrix.example.com:443"
}
```

Optionally these files can be served as json. Therefore you can set the following in the VirtualHost of example.com:

```
<Location "/.well-known/matrix/server">
        Header always set access-control-allow-origin "*"
        Header always set access-control-allow-methods "GET, HEAD, OPTIONS"
        Header always set access-control-allow-headers "Origin, X-Requested-With, Accept, Date"
        Header always set Content-Type "application/json; charset=UTF-8"
</Location>
<Location "/.well-known/matrix/client">
        Header always set access-control-allow-origin "*"
        Header always set access-control-allow-methods "GET, HEAD, OPTIONS"
        Header always set access-control-allow-headers "Origin, X-Requested-With, Accept, Date"
        Header always set Content-Type "application/json; charset=UTF-8"
</Location>
```

Now you can test your installation at https://federationtester.matrix.org.


## Register new user

```bash
register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml http://localhost:8008
```

## Use LDAP for authentification

This section is based on the corresponding section of https://romangeber.com/matrix_synapse_on_arch_linux/.

### Installation

```bash
apt install matrix-synapse-ldap3
```

### Configuration

```
password_providers:
 - module: "ldap_auth_provider.LdapAuthProvider"
   config:
   enabled: true
       uri: "ldap://localhost:389"
       start_tls: false
       base: "ou=users,dc=example,dc=com"
       attributes:
           uid: "cn"
           mail: "mail"
           name: "givenName"
       #bind_dn:
       #bind_password:
       #filter: "(objectClass=posixAccount)"
```

## Setup TURN (for Video- and Audio-Conference)

### Installation

```bash
apt install coturn
```

### Configuration

This configuration file is /etc/turnserver.conf:
```
use-auth-secret
static-auth-secret="secretKey" (generate a key, e.g. via `pwgen -s 64 1`)
realm=example.com
secure-stun
mobility
cert=/etc/letsencrypt/live/example.com/cert.pem
pkey=/etc/letsencrypt/live/example.com/privkey.pem
no-multicast-peers
no-tlsv1
no-tlsv1_1
```

Configure the Matrix server to use turn, in homeserver.yaml:
```
turn_uris:
 - "turns:turn.example.com?transport=udp"
 - "turns:turn.example.com?transport=tcp"
 - "turn:turn.example.com?transport=udp"
 - "turn:turn.example.com?transport=tcp"

turn_shared_secret: "secretKey" (see above)
```

Don't forget to configure the [Firewall](/firewall) accordingly.

# Optional Element Web

Download the latest (stable) relase of Element Web from https://github.com/vector-im/element-web/releases, extract to the webroot of chat.example.com and make sure the files are owned by root:root.

The configuration file for Element Web is config.json. You might want to copy the config.sample.json and configure

```
default_server_config -> m.homeserver -> base_url: https://matrix.example.com
brand: example Matrix
jitsi -> preferredDomain -> <Url of your Jitis Instance>
```

Furthermore configure to serve these files via

```
<VirtualHost _default_:80>
        ServerName chat.example.com
        ServerAlias chat.example.com
        Redirect permanent / https://chat.example.com/
</VirtualHost>

<VirtualHost _default_:443>
        ServerName chat.example.com
        ServerAlias chat.example.com
        DocumentRoot /var/www/html/chat_example_com
        Protocols h2 http/1.1
        SSLEngine on
        SSLCertificateFile    /etc/letsencrypt/live/example.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
        # HSTS einrichten -- erfordert mod_headers!
        Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains"
        Header always set X-Frame-Options: SAMEORIGIN
        Header always set X-Content-Type-Options: nosniff
        Header always set Content-Security-Policy: "frame-ancestors 'none'"
        Header always set X-XSS-Protection "1; Modus = Block"
        Header always set Referrer-Policy: same-origin
        <Directory /var/www/html/chat_example_com>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>
</VirtualHost>
```