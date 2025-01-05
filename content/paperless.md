---
title: "Paperless-ngx"
date: 2024-12-18T22:04:05+01:00
---

## Overview

Projectpage: https://docs.paperless-ngx.com.

This article descibes our paperless-ngx setup using their docker setup with PostgreSQL as the database backend.

## Installation & Configuration

While following the [setup page of the project](https://docs.paperless-ngx.com/setup/#docker) and choosing PostgreSQL as the database backend adjust `docker-compose.env` as needed, e.g.
```
PAPERLESS_APP_TITLE=my-paperless-instance

# Run paperless as root (in the container) aka. the user running podman (outside the container).
# This will ensure the volume folder are owned by the user starting the container.
USERMAP_UID=0
USERMAP_GID=0

# This is required if you will be exposing Paperless-ngx on a public domain
# (if doing so please consider security measures such as reverse proxy)
PAPERLESS_URL=https://paperless.example.com

# Adjust this key if you plan to make paperless available publicly. It should
# be a very long sequence of random characters. You don't need to remember it.
PAPERLESS_SECRET_KEY=my-super-secret-pass

# Use this variable to set a timezone for the Paperless Docker containers. Defaults to UTC.
PAPERLESS_TIME_ZONE=Europe/Berlin

# The default language to use for OCR. Set this to the language most of your
# documents are written in.
PAPERLESS_OCR_LANGUAGE=deu

# As we use podman to check for new versions, we disable the update check in paperless.
PAPERLESS_ENABLE_UPDATE_CHECK=false

# File name handling
# By default, paperless stores your documents using the identifier which it has assigned to each document. 
# To let paperless use a folder structure, configure something like the following,
# compare https://docs.paperless-ngx.com/advanced_usage/#file-name-handling
PAPERLESS_FILENAME_FORMAT={{ created_year }}/{{ correspondent }}/{{ title }}

# Une more than one task to parse documents
PAPERLESS_TASK_WORKERS=4

# Consume documents in subfolders
PAPERLESS_CONSUMER_RECURSIVE=true
```

## Useful configuration

Here are some setup options that might be useful:
- After creating users, create a tag for every user that is learned automatically and configure the tag to set the unix file owner and permission accordingly.
- Paperless supports an `Inbox` tag. This is usefully to not let new documents disappear in all documents with possibly wrong metadata. Let paperless set this tag for every incomming document and remove it manually after checking the metadata.

## Tips & Tricks

### Sanity checker

Paperless-ngx has a [build-in sanity checker](https://docs.paperless-ngx.com/administration/#sanity-checker) that inspects your document collection for issues:
```bash
document_sanity_checker
```
