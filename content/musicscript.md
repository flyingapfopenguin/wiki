---
title: "Music Script"
date: 2023-04-06T22:54:01+02:00
---

We use the following scipt to play audio files in one folder (randomly ordered) from the command line.

Note: The Bash script uses mplayer.

Script:

```bash
#!/bin/bash

folder=${1:-"."}
extension=${2:-"mp3"}
search_string=${3:-""}
cur_folder=${pwd}

cd "${folder}"
mplayer -shuffle *${search_string}*.${extension}
cd "${cur_folder}"
```

Usage, where

* folder (optional, default: ".") is the path of the folder with the audio files,
* extension (optional, default: "mp3") is file extension the list of files is filtered and
* search_string (optional, default: "" (none)) is a string in the file name the list of files is filtered, e.g. a name of a band:

```bash
./backup.sh folder extension search_string
```