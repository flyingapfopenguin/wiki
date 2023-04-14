---
title: "Julia"
date: 2023-04-06T22:54:01+02:00
---

## Installation

Clone the git repository https://github.com/JuliaLang/julia.git, checkout the desired version, install the dependencies
```bash
apt install make gcc g++ gfortran
```

 and build Julia:
```bash
make clean; make
```

Remember to delete ~/.julia before the first start of julia for a clean setup.

Our setup includes the following packages:

* Nemo
* Hecke
* Coverage
* GAP

To update the installed packages and install the develop version of a package, open the package manager with ] in julia and run
```
update
dev <package>
```

Remember to install the GAP dependencies see [GAP Installation](/gap#installation).