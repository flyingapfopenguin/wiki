---
title: "Strings"
date: 2023-04-06T22:54:01+02:00
---

## Compare two strings

We use the following script to compare two strings

```bash
#!/bin/bash
diff <(echo "$1") <(echo "$2")
```