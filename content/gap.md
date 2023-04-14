---
title: "GAP"
date: 2023-04-06T22:54:01+02:00
---

## Installation

Clone the git repository https://github.com/gap-system/gap.git and follow the instructions on the GitHub page.

Additionally to the dependencies listed on the GitHub page, we need the following packages to build all packages:

* boost library (for NormalizInterface)
* C binding for ZeroMQ (for ZeroMQInterface)
* curl library (for curlInterface, see https://www.gap-system.org/Manuals/pkg/curlInterface-2.1.1/doc/chap1.html)
* Cddlib (for CddInterface, see https://github.com/kamalsaleh/CddInterface)

To install those on debian run
```bash
apt install libboost-all-dev libczmq-dev libcurl4-gnutls-dev libcdd-dev
```

## Configuration

A user-specific configuration for gap can be saved in ~/.gap/gap.ini.

To generate this config file with the default values (all commented out) simply run:

```
gap> WriteGapIniFile();
```

To, e.g., change the automatically loaded packages, use the commands from [the documentation](https://www.gap-system.org/Manuals/doc/ref/chap76.html#X7E6767B485F23BFC), [source](https://www.gap-system.org/Manuals/pkg/loops/doc/chap1_mj.html#X8360C04082558A12):

```
gap> pref := UserPreference( "PackagesToLoad " );;
gap> Add( pref, "packageA" );;
gap> Add( pref, "packageB" );;
gap> Add( pref, "packageC" );;
gap> SetUserPreference( "PackagesToLoad", pref );;
gap> WriteGapIniFile();;
```

## Load default gap packages automatically under Arch

Since version 4.8.6 core gap and the extended gap packages are split up into two arch packages: gap and gap-packages. Since then one might only have core gap installed, so gap only loads these packages per default, see the [gap-no-packages-by-default.patch](https://git.archlinux.org/svntogit/community.git/tree/trunk/gap-no-packages-by-default.patch?h=packages/gap) in the arch gap package.

If you have gap-packages installed, you can set the list of automatically loaded packages back to the original default
```
[ "autpgrp", "alnuth", "crisp", "ctbllib", "factint", "fga", "irredsol", "laguna", "polenta", "polycyclic", "resclasses", "sophus", "tomlib" ].
```

To do so, run the following code (compare Configuration):
```
gap> SetUserPreference( "PackagesToLoad", [ "autpgrp", "alnuth", "crisp", "ctbllib", "factint", "fga", "irredsol", "laguna", "polenta", "polycyclic", "resclasses", "sophus", "tomlib" ] );;
gap> WriteGapIniFile();;
```

## Short Reference

### Minimal polynomial and Field extensions

To determine the degree of Q(Sqrt(2),Sqrt(1+i))/Q, we construct Q(Sqrt(1+i)) and examine the Minimal polynomial of Sqrt(2) over that field:

```
gap> x:=Indeterminate(Rationals, "x");;
gap> Factors((x^2-1)^2+1); # MiPo of Sqrt(1+i) over Q?
[ x^4-2*x^2+2 ]
gap> k:=AlgebraicExtensionNC(Rationals, (x^2-1)^2+1, "x"); # Q(Sqrt(1+i))
<algebraic extension over the Rationals of degree 4>
gap> y:=Indeterminate(k,"y");;
gap> Factors(y^2-2); # irreducibel over k=Q(Sqrt(1+i))?
[ y^2+(!-2) ]
gap> Factors(y^2+1); # ... but i is element of k=Q(Sqrt(1+i))
[ y+(x^2-1), y+(-x^2+1) ]
```