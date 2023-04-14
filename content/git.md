---
title: "git"
date: 2023-04-06T22:54:01+02:00
---

## Installation

```bash
apt install git # on Debian
emerge -av dev-vcs/git # on Gentoo
pacman -S git # on Arch
```

## Configuration

Set Name and Mail-Address:

```bash
git config [--global] user.name <Name>
git config [--global] user.email <email>
```

## Tipps & Tricks

tig is a usefull tool to quickly scroll through commits.

[This website](https://gitignore.io) is usefull to generate *.gitignores*.

In [this repo](https://github.com/github/training-utils) Matthew McCullough collected some nice tools for git training, e.g. a command to generate n commits at once.

### Run fsck on the git object store

```bash
git fsck --full --strict --progress
```

### Show different file states during a merge (conflict)

```bash
git show :0:<file> # currently staged version
git show :1:<file> # version of the common ancestor
git show :2:<file> # version of the current branch (HEAD)
git show :3:<file> # version of the merged branch
```

### Continuously watch log

```bash
watch -n2 -t -c git -c color.diff=always log --graph --decorate --all
```

## Underneath

[source](https://www.youtube.com/watch?v=ig5E8CcdM9g)

### Parse short rev etc.

To parse any "label", e.g. a short rev, a branch name, a tag, etc. to complete hash run
```bash
git rev-parse <string>
```

### Reading a git object manually

To read a git object without git at all, one can use perl:

```bash
perl -MCompress::Zlib -e 'undef $/; print uncompress(<>)' .git/objects/<hash>
```

Or using git, to determine the type of the object and get the content:

```bash
git cat-file -t <hash>
git cat-file -p <hash>
```

### Create a git hash of a blob manually

```bash
echo "Hello World" | git hash-object -w --stdin
```

### Determine merge base

```bash
git merge-base <ref1> <ref2> ... <refn>
```
