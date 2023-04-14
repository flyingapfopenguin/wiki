---
title: "Gentoo"
date: 2023-04-06T22:54:01+02:00
---

## Update, Cleaning & Rebuild

Update:

```bash
etc-update                                                                                                                                                   
eix-sync
emerge -avuNDt --with-bdeps=y --keep-going=y --verbose-conflicts --changed-deps @world
emerge -av1 @preserved-rebuild
emerge -ac
etc-update
```

Cleaning:

```bash
./update
eix-test-obsolete ### handle the output manually
portpeek -aq
perl-cleaner --reallyall
eclean -d distfiles -f
eclean -d packages
### /var/tmp/portage clean?
```

Rebuild:

```bash
./cleaner # including ./update                                                                                                                                     
emerge -v sys-devel/gcc
emerge -ev --keep-going=y @system
emerge -ev --keep-going=y @world
# Kernel rebuild
```

While the above commands rebuild @system and @world in one run, that cannot be interrupted, we might want to use the following script to rebuild the system.
The script emerges all installed packages that have not been build after the last merge of the gcc compiler.
Please note that it assumes, you have exactly one version of the gcc compiler installed.

```bash
#!/bin/bash
gccWithVersion=$(eix sys-devel/gcc --format '<installedversions:NAMEASLOT>' | grep "sys-devel/gcc:")
if [ $(echo "$gccWithVersion" | wc -l) != 1 ]; then
  echo "seems we found more than one gcc installed"
  exit 1
fi
packageListLastCompileBeforeGCC=$(eix '-I*' --format '<installedversions:DATESORT>' | sort -n | cut -f3 | grep -B 100000 $gccWithVersion | head -n -1)
nbPackages=$(echo -n "$packageListLastCompileBeforeGCC" | wc -l)
echo "found $nbPackages package(s) to rebuild"
if [ $nbPackages = 0 ]; then
  exit 0
fi
packageListEmerge=$(echo "$packageListLastCompileBeforeGCC" | tr '\n' ' ')
emerge -av1 --keep-going=y $packageListEmerge
```

## Kernel Update

```bash
eselect kernel set <n>
cd /usr/src/linux
cp ../linux-<oldVersion>-gentoo/.config .
make oldconfig
make clean && make all -j3 && make modules_install && emerge -v1 @module-rebuild
cp arch/x86/boot/bzImage /boot/kernel-<thisVersion>-gentoo
# build new initramfs
```