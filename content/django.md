---
title: "Django"
date: 2023-04-06T22:54:01+02:00
---

see: https://www.djangoproject.com.

## Deployment on a Debian Server

This section was tested on Debian 9 (stretch) with Django 2.2.

### Create virtual env and install Django

see [Python-venv](/python#python-venv).

For the rest of this section, we assume that the virtual env is located at /var/www/myvenv and the django project is located at /var/www/mysite.
All these files should be owned by root.

### Deactive Debug Mode

mysite/mysite/settings.py:
```
DEBUG = False
```

### Add domain to allowed hosts

mysite/mysite/settings.py:
```
ALLOWED_HOSTS = [ '<domain>' ]
```

### Configure database

For installation und initialization see [MariaDB](/mariadb).

To configure the database for django edit mysite/mysite/settings.py:
```
DATABASES = {
    'default': {       
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '<database>',
         'USER': '<djangoDatabaseUser>',
         'PASSWORD': '<djangoDatabaseUserPassword>',
         'HOST': 'localhost',
         'PORT': '',
         'OPTIONS': {
             'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
         },
    }
}
```

To initialize the database, we first need the install gcc:
```bash
apt install build-essentiall
```

Afterwards we install a database client and initilize the database via (in the virtual env):
```bash
pip install mysqlclient
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

### Configure static files

To configure the path for static files add to mysite/mysite/settings.py:
```
STATIC_ROOT = '/var/www/mysite/static/'
```

and let django collect the necessary static files from the django installation via (in the virtual env):
```bash
python manage.py collectstatic
```

### Setup Apache configuration

We need WSGI, so
```bash
apt install libapache2-mod-wsgi libapache2-mod-wsgi-py3
```

Use the default SSL configuration for apache, see [Apache](/apache#configuration).

Edit the configuration as follows. Add the configuration for WSGI, edit the directory configuration and add the configuration for static files:
```
WSGIDaemonProcess <domain> processes=2 threads=15 display-name=%{GROUP} python-home=/var/www/myvenv python-path=/var/www/mysite
WSGIProcessGroup <domain>
WSGIScriptAlias / /var/www/mysite/mysite/wsgi.py
<Directory /var/www/mysite>
  Require all granted
</Directory>
Alias /static/ /var/www/mysite/static/
```
