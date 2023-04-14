---
title: "RAW Converter to jpg"
date: 2023-04-06T22:54:01+02:00
---

We use the following script to convert Canon RAW files (CR2) to jpg and additionally resize them to space saving smaller jpgs.

Plase note that you can adapt the two parameters "scale" and "quality" for the smaller jpgs.

```bash
#/bin/bash
for i in *.CR2; do dcraw -c -e $i > `basename $i .CR2`.jpg; convert -scale 33% -quality 75 `basename $i .CR2`.jpg `basename $i .CR2`_small.jpg; echo -n "."; done
```