---
title: "LaTeX"
date: 2023-04-06T22:54:01+02:00
---

## Installation

To install texlive:

```bash
apt install texlive-full latexmk # on Debian
emerge -av app-text/texlive dev-texlive/texlive-bibtexextra dev-texlive/texlive-fontsextra dev-texlive/texlive-humanities dev-texlive/texlive-latexextra dev-texlive/texlive-luatex dev-tex/latexmk # on Gentoo
yay -S texlive-core texlive-bibtexextra texlive-fontsextra texlive-humanities texlive-latexextra latex-mk # on Arch, latex-mk is from the AUR
```

To manually install single packages place them at:

```
/usr/share/texmf/tex/latex/<packageName> # for system-wide installation
~/texmf/tex/latex/<packageName> # for user specific installation
```

## Makefile, when-changed & gitignore

We use the following Makefile to build LaTeX projects

```
PROJNAME=<name> # TODO
OUT_DIR=out

LATEXC = latexmk
LATEXFLAGS = -outdir=$(OUT_DIR) -pdf -halt-on-error -use-make -file-line-error -shell-escape

all: dir $(PROJNAME).pdf

dir:
	mkdir -p $(OUT_DIR)
	mkdir -p $(OUT_DIR)/chapters

$(PROJNAME).pdf: $(PROJNAME).tex
	$(LATEXC) $(LATEXFLAGS) $<
	cp $(OUT_DIR)/$(PROJNAME).pdf $(PROJNAME).pdf

clean:
	rm -rf $(OUT_DIR)
	rm $(PROJNAME).pdf

open: $(PROJNAME).pdf
	evince $(PROJNAME).pdf

.PHONY: all dir clean
.SILENT: dir clean
```

The following script automatically rebuilds the project from scretch, when a file in the project folder or the "section" subfolder changes.

```bash
apt install inotify-tools # on Debian
emerge -av sys-fs/inotify-tools # on Gentoo
pacman -S inotify-tools # on Arch
```

```bash
#! /bin/bash

path=$1;
paths=$path
[ -d "$path/sections" ] && paths+=" $path/sections"

while inotifywait -q -e close_write,moved_to,create $paths; do
	( cd $path && make clean; make all )
done
```

Based on your folder structure the following .gitignore is appropriate

```
out/*
<name>.pdf # TODO
```